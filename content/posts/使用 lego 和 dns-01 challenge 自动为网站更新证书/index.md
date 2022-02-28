---
title: "使用 lego 和 dns-01 challenge 自动为网站更新证书"
date: 2022-01-15T19:14:00+08:00
Tags: ['blog']
---

## Lego

[https://github.com/go-acme/lego](https://github.com/go-acme/lego)

Lego 是一个使用 Golang 编写的 ACME 客户端，主要适用于 Let's Encrypt 颁发的证书，lego 实现了多个 DNS 服务商的 API 接口，可以非常方便的使用 DNS-01 Challenge 来生成证书。同时 LEGO 不经支持使用 cli 接口，还可以作为 Golang lib 来使用，可以说是非常方便了。

## Lego 与 Cloudflare

Lego 更新 DNS 记录需要使用 Cloudflare 的 API，所以我们需要提供相应的 Token。下面记录一下申请 Token 的过程

1. 首先打开 Cloudflare 的 API 令牌界面

    ![Untitled](%E4%BD%BF%E7%94%A8%20lego%20%E5%92%8C%20%20d8ab5/Untitled.png)

2. 点击创建令牌

    ![Untitled](%E4%BD%BF%E7%94%A8%20lego%20%E5%92%8C%20%20d8ab5/Untitled%201.png)

3. 选择使用“编辑区域 DNS”模板
4. 权限按如下设置

    Lego 在申请证书时不仅会使用编辑 DNS 的权限，还需要读取“区域”的权限才可以正常使用

    ![Untitled](%E4%BD%BF%E7%94%A8%20lego%20%E5%92%8C%20%20d8ab5/Untitled%202.png)

5. 创建完成

![Untitled](%E4%BD%BF%E7%94%A8%20lego%20%E5%92%8C%20%20d8ab5/Untitled%203.png)

## Lego 设置

[Examples](https://go-acme.github.io/lego/usage/cli/examples/)

[Cloudflare](https://go-acme.github.io/lego/dns/cloudflare/)

Lego 的设置非常简单，只需要把刚才的令牌设置为环境变量 `CLOUDFLARE_DNS_API_TOKEN` 就可以了。Lego 生成的证书会保存在命令运行的目录中的 `.lego` 文件夹中

为了方便可以使用 docker-compose 来使用

```yaml
lego:
image: goacme/lego:latest
entrypoint: ["lego","--email=example@example.com", "--dns", "cloudflare", "-d", "lyine.pw", "-d", "*.lyine.pw", "-a", "renew", "--days", "45"]
env_file:
  - ./lego/env.lego
volumes:
  - ./lego/www/:/.lego/
```

上面提到的令牌以 `env_file` 的形式保存在 `./lego/env.lego` 中，`-a` 参数可以默认同意 Let's Encrypt 的用户协议，`--day 45` 表示如果距离证书过期小于 45 天就更新证书。

<aside>
💡 Lego 命令的格式为 `lego [global options] command [command options] [arguments...]` 也就是说 command 也就是 `run` `renew` 命令的位置需要在 `[global options]` 之后

</aside>

## crontab 自动化

使用 crontab 使得更新证书的过程可以自动化

```bash
## renew ssl certificate using lego every 15days
0 5 */15 * * cd /root/ && docker-compose up lego >> /root/lego/lego.log 2>&1
```

使用 [https://crontab.guru/](https://crontab.guru/) 这个网站可以验证自己的 crontab 表达式是否正确，crontab 输出的问题在 后面的回答中有提到

[How to redirect output to a file from within cron?](https://unix.stackexchange.com/questions/52330/how-to-redirect-output-to-a-file-from-within-cron)

最后记得定时运行 `nginx -s reload` 来更新证书的缓存。