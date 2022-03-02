---
title: "arch linux 英伟达驱动折腾"
date: 2021-09-30T10:54:00+08:00
tags: ['linux', 'nvidia', '驱动']
---

首先就是安装驱动，要注意的就是一定要把系统和其他依赖都一起升级到最新不然死活都在用nouveau，就算写了 blacklist也不管用

然后要注意在笔记本电脑上一般是使用intel和nvidia的混合显卡，而网上的教程一般都是只有一个显卡（独立显卡）的情况，使用 nvidia-xconfig 生成的 Xorg.conf 也是如此。如果按一般的教程来走的话，最后会卡在开机，变成黑屏，只能 ctrl alt f2切换到其他 tty 来操作，打开Xorg log会发现有 fail to load glxserver_nvidia 报错。

一般教程 [https://wiki.archlinux.org/title/NVIDIA_(简体中文)](https://wiki.archlinux.org/title/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

解决方式是在bios里禁用intel显卡

当然也有继续使用混合显卡的方法在arch wiki上

[https://wiki.archlinux.org/title/NVIDIA_Optimus_(简体中文)](https://wiki.archlinux.org/title/NVIDIA_Optimus_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

但这个方法没有用

## 解决方法：optimus-manager

<https://github.com/Askannz/optimus-manager>

但要注意在项目 wiki 中作者提到如果使用 steam play 等使用 Vulkan 的程序需要在环境变量中添加

```bash
VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json
```

> **My Vulkan applications/Steam Play/DXVK games do not work in Nvidia mode**
>
> If you have Vulkan drivers installed for both your Intel and Nvidia GPUs, it can happen that applications try to use the Intel GPU even though your desktop is switched to the Nvidia one. This causes a crash since the Intel GPU is not actually available. To work around this, set the environment variable `VK_ICD_FILENAMES=/usr/share/vulkan/icd.d/nvidia_icd.json` before starting your application.


