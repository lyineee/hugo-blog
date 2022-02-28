---
title: "设置使用rsa-key登陆树莓派"
date: 2021-03-16T10:31:00+08:00
tags: ['raspberrypi', 'ssh']
---

运行 `ssh-keygent -t rsa`  ,根据提示生成.pub公钥和私钥

## 树莓派

在树莓派新建 ~/.ssh/authorized_keys，并将公钥的内容复制到这个文件中

编辑 `/etc/ssh/sshd_config` 文件，加入

```bash
PubkeyAuthentication yes
AuthorizedKeysFile ~/.ssh/authorized_keys
```

保存，重启 sshd 就可以了

💡 注意：.ssh 目录的权限应该为700，authorized_keys 文件权限应为 600

## 电脑

把私钥复制到 ~/.ssh/ 目录中

编辑 ~/.ssh/config 文件，加入

```bash
# raspberry pi
Host pi
HostName raspberrypi
User pi
IdentityFile ~/.ssh/raspberrypi
```

现在就可以使用 `ssh pi` 来连接树莓派了