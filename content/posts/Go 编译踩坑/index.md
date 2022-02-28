---
title: "Go 编译踩坑"
date: 2021-03-16T10:11:00+08:00
tags: ['compile', 'go']
---

#### 首先是出现 loadinternal: cannot find runtime/cgo 的问题

One way to avoid this issue is to force a static build disabling cgo. Try building your binary with the following command:

`RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o yourBinaryName`

If you actually need cgo in your app, you might want to install gcc, musl-dev and even build-base like @abergmeier pointed out. You can do this with this command:

`RUN apk update && apk add --no-cache musl-dev gcc build-base`

If none of that works I would check out this gist claiming it creates a Dockerfile capable of compiling protobuffs: ["Alpine (3.6) based docker image with Protocol Buffers compiler supporting Go"](https://gist.github.com/rizo/513849f35178d19a13adcddf2d045a19)

From <[https://stackoverflow.com/questions/55932233/cannot-find-runtime-cgo-in-alpine](https://stackoverflow.com/questions/55932233/cannot-find-runtime-cgo-in-alpine)>

#### 在安装时出现 /x 的包i/o timeout 的问题

```bash
go env -w
GO111MODULE=on
go env -w
GOPROXY=https://goproxy.cn,direct
```

#### apk add go gcc libc-dev

最开始没有加上 libc-dev的时候直接运行go build会报错

![Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/1.png](Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/1.png)



网上查缺这几个文件要安装的依赖是libc6-dev，但apk的包里没有，所以改成了libc-dev然后就可以编译成功了

很多包都有对应的加 -dev 的包（用于提供动态编译库？）

发现其实最好的办法是直接clone github上的项目，然后在项目内执行 go

在其他系统上运行编译出的文件时出现

![Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/2.png](Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/2.png)

出现这个问题的原因是编译不会打包静态文件，所以web的内容丢失了

> 对于 Go 语言开发者来说，在享受语言便利性的同时，最终编译的单一可执行文件也是我们所热衷的。但是，Go在编译成二进制文件时并没有把我们的静态资源文件编译进去，如果我们开发的是web应用的话就需要想办法把我们的静态文件也编译进去。
 From [https://c.isme.pub/2019/01/10/go-static/](https://c.isme.pub/2019/01/10/go-static/)
>

在BaiduPCS-Go的github页面的交叉编译帮助中看到了

> 将网页静态文件打包进程序
地址: [https://github.com/GeertJohan/go.rice](https://github.com/GeertJohan/go.rice)
打包命令:
`rice -i github.com/iikira/BaiduPCS-Go/internal/pcsweb append --exec BaiduPCS-Go`
From [https://github.com/iikira/BaiduPCS-Go/wiki/编译-交叉编译帮助](https://github.com/iikira/BaiduPCS-Go/wiki/%E7%BC%96%E8%AF%91-%E4%BA%A4%E5%8F%89%E7%BC%96%E8%AF%91%E5%B8%AE%E5%8A%A9)
>

在安装go.rice后使用 `rice append -i ./BaiduPCS-Go/internal/pcsweb --exec /root/main` 就可以导入静态文件了

有很多商业软件在发布时按照LSB规范来分发二进制。如果系统不符合LSB规范则这些程序都无法运行，比如下面这个过程

```bash
$ ./lmgrd
bash: ./lmgrd: No such file or directory
```

查看文件存在

```bash
$ file ./lmgrd
./lmgrd: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```

执行权限也正确

```bash
$ ls -lha ./lmgrd
 rwxr-xr-x 1 em em 854K Jun 8 2009 ./lmgrd
```

用LDD检查，无法查看内容

```bash
$ ldd ./lmgrd
/usr/bin/ldd: line 116: ./lmgrd: No such file or directory
```

From
<[https://tieba.baidu.com/p/2415224308?red_tag=2037124312](https://tieba.baidu.com/p/2415224308?red_tag=2037124312)>

## Docker 镜像构建

想使用docker的autobuild功能来云构建docker image，但发现一些构建的问题

1. 在云端构建的系统是amd64架构的，所以在构建arm架构的镜像时要交叉编译，对于golang来说还是很简单的
2. 一些不标注系统架构的镜像会自动使用构建机器，也就是云端的机器架构的镜像（amd64）。所以RUN这个命令是不能使用的
3. docker并不是虚拟机，所以对于交叉编译的可执行文件，有时也是可以在不同架构的系统上运行的

![Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/3.png](Go%20%E7%BC%96%E8%AF%91%E8%B8%A9%E5%9D%91%203b2cb/3.png)

对于这个dockerfile云构建的镜像（amd64），再去掉ENTRYPOINT后在arm下是可以运行的，原因就是 main 这个可执行文件是针对arm交叉编译而来的，所以可以在arm架构的系统上运行，即使这个镜像的base image是amd64架构的，这说明容器是依赖于宿主机的部分环境的。**所以在跨平台构建docker镜像是要注意base image架构的选择，同时可以使用多阶段构建（dockerfile multi-stage
builds）**，区分编译镜像和运行镜像