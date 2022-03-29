---
title: "Kubernetes and Flannel with Wireguard"
date: 2022-03-29
tags: ['kubernetes', 'linux', 'network']
---
## k8s 通信模型

- 运行在一个节点当中的Pod能在不经过NAT的情况下跟集群中所有的Pod进行通信
- 节点当中的客户端（system daemon、kubelet）能跟该节点当中的所有Pod进行通信
- 以host network模式运行在一个节点上的Pod能跟集群中所有的Pod进行通信

k8s 通信模型的实现是交给其他应用的完成的，如 k3s 就自带了 flannel 来实现这个模型，接下来也会以 flannel 的实现来作为例子。

## Flannel

 由于阿里云的轻量应用服务器的公网ip实在NAT后的，所以在 ifconfig 中只有内网ip，这会导致 Flannel 获取内网ip，使得不是一个云服务商的ECS不能组成集群，跨节点的pod通信也不能进行。所以需要更改 Flannel 获取的公网ip(public-ip)，这个 public-ip 可以在 node annotation 中看到

```bash
Annotations: 
  http://flannel.alpha.coreos.com/backend-data: "sz1skDooVieIrwLInioYwItM1MfCUZI0iz8/0kioeR0=" 
  http://flannel.alpha.coreos.com/backend-type: extension [flannel.alpha.coreos.com/kube-subnet-manager:](http://flannel.alpha.coreos.com/kube-subnet-manager:) true
  http://flannel.alpha.coreos.com/public-ip: 104.156.140.173
```

要更改这个 public-ip 可以通过增加一个 annotation 来做到 `kubectl annotate nodes <master> flannel.alpha.coreos.com/public-ip-overwrite=<master_pub_ip>` 。添加后重启 k3s 即可 `systemctl restart k3s`

## Flannel with Wireguard

k3s Flannel 默认使用 VXLAN 后端，可能会有安全问题，且可能对于有NAT的情况不友好。所有可以使用 Wireguard 来代替以获得更好的效果。

在 master 节点上添加 k3s 启动参数 `-flannel-backend=wireguard` 或在 `/etc/rancher/k3s/config.yaml` 中添加一行 `flannel-backend: wireguard` 以使用 Wireguard 代替 VXLAN 后端。

在每一个节点上都要安装 Wireguard 对于 Ubuntu `sudo apt update && sudo apt install wireguard` 来安装，安装完成后重启 k3s 通过 `wg show` 来查看 wireguard interface

```bash
$ wg show
interface: flannel.1
  public key: BWaYu5kACYCD4Rs7ukfbq5x9L5ERvW6BQ5bQkcfxpUQ=
  private key: (hidden)
  listening port: 51820
```

并可以使用 `netstat -ulnp` 来查看 Wireguard 的 UDP 端口占用

```bash
$ netstat -ulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
udp        0      0 127.0.0.53:53           0.0.0.0:*                           387/systemd-resolve
udp        0      0 172.17.61.146:68        0.0.0.0:*                           255/systemd-network
udp        0      0 0.0.0.0:51820           0.0.0.0:*                           -
udp6       0      0 :::51820                :::*                                -
```

可以看到 Wireguard 会使用 51820 端口进行通信，而后面的 `PID/Program name` 为 `-` 则说明这是从内核中的调用（Wireguard 内核模块）

> 💡 使用 VXLAN 后端时也可以使用相同的命令来查看 UDP 协议端口的占用

在有节点加入后使用 `wg show` 可以看到有新的 peer 产生

```bash
$ wg show
interface: flannel.1
  public key: BWaYu5kACYCD4Rs7ukfbq5x9L5ERvW6BQ5bQkcfxpUQ=
  private key: (hidden)
  listening port: 51820

peer: sz1skDooVieIrwLInioYwItM1MfCUZI0iz8/0kioeR0=
  endpoint: 104.156.140.173:51820
  allowed ips: 10.42.1.0/24
  latest handshake: 2 minutes ago
  transfer: 3.93 KiB received, 4.06 KiB sent
  persistent keepalive: every 25 seconds
```

> 💡 如果有节点不能通过 wireguard 连接，Flannel 会不断尝试连接产生 peer，最后回导致 `wg show` 中有很多 peer，这时可以通过命令 `for i in $(wg |grep peer|awk '{print $2}');do wg set flannel.1 peer $i remove;done` 来删除所有 peer

## Wireguard 的问题

再将一个节点加入集群后发现这一个节点不能与控制面板所在的节点通信，且 `wg show` 命令发现没有任何的输出，`lsmod | grep wireguard` 发现 wireguard 内核模块没有加载，`modprobe wireguard` 报错 `FATAL: Module wireguard not found in directory` 。自己建立一个 `wg0.conf` 尝试 `wg-quick up wg0` 报错 `RTNETLINK answers: Operation not supported` 按 [stackoverflow](https://stackoverflow.com/questions/62356581/wireguard-vpn-how-to-fix-operation-not-supported-if-it-worked-before) 上的答案尝试更新 linux-header 发现问题依旧存在。最后发现有问题的这个节点的 linux 内核版本 `4.15.0-20-generic` 与其他节点版本 `4.15.0-162-generic` 相比很低，最后更新内核 `sudo apt update && sudo apt apt dist-upgrade` 后 Wireguard 工作正常了。

> ❓最后解决问题时还将 proxier 从 iptables 改为了 ipvs 模式`-kube-proxy-arg "proxy-mode=ipvs" "masquerade-all=true"` 不知道与问题的解决有没有关系
