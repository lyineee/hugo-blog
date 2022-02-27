---
title: "在windows上调试safari"
date: 2021-03-16T9:53:10+08:00
tags: ['ios', 'web']
---

因为手上用的ios的设备，所以网页也要支持safari才行，不过因为是移动设备的关系，和电脑上chrome调试的网页还是有所差别的，主要的一个问题就是移动设备的页面大小是不停变化的，而且在safari上css的vh这个单位的表现似乎和windows上的有所不同，具体表现为在safari的地址栏和最下方的工具栏大小有变化时，vh的大小并没有相应的发生变化，导致了网页元素的一些互相遮挡。而我手里没有Mac OS的设备，一时间不知道应该如何调试safari上的网页，不过上网查了查还真有这么个东西。

## [ios-webkit-debug-proxy](https://github.com/google/ios-webkit-debug-proxy)

这个软件可以使用ios safari的网页检查器(Web Inspector)，来远程连接safari从而来进行调试

### 安装

使用scoop

```bash
scoop bucket add extrasscoop install ios-webkit-debug-proxy
```

### 使用

1. 安装iTunes
2. 从ios上safari的设置中打开网页检查器（设置->safari浏览器->高级->网页检查器）
3. 将设备连接到电脑上，在ios设备上选择信任设备
4. 启动`ios-webkit-debug-proxy`，在弹出的windows的安全警报中允许在所有网络中通信，然后有如下的输出
    ![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152448.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152448.png)
5. 现在在safari中打开一个网页，再在电脑上打开网页 `localhost:9221`， 就可以看到设备了，点击就可以看到你在safari中打开的网页了。
    ![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152504.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152504.png)

但这时点击连接后也不会有调试工具供我们使用，还要安装adapter来使用chrome的调试工具

## [remotedebug-ios-webkit-adapter](https://github.com/RemoteDebug/remotedebug-ios-webkit-adapter)

这个就是对应chrome调试工具的转换器，它会在运行时自动启动ios-webkit-debug-proxy，非常方便。

### 安装转换器

首先要安装 ios-webkit-debug-proxy， 然后使用npm来安装 `npm install remotedebug-ios-webkit-adapter -g`

### 使用转换器

直接运行这个程序，它在默认的端口(9000)运行，但如果出现端口占用的报错，可以使用 –port 参数来设置运行的端口号 在chrome中打开

```bash
chrome://inspect
```

，会看到如下页面

![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152516.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152516.png)

在Discover network targets后面的Configure中

![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152527.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152527.png)

添加刚刚运行程序的端口，如：

```bash
localhost:9000
```

(默认端口) 点击Done后，等待一小会，
**不知道为什么反正这里就是要等一会才会出现东西，就会出现下面的东西**

![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152551.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152551.png)

点击对应网页下面的inspect就可以看到熟悉的chrome dev工具啦

![https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152558.png](https://raw.githubusercontent.com/lyineee/imgbed/master/20200824152558.png)
