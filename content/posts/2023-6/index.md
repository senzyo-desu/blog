---
title: "Arch与Intel核显"
date: 2023-04-07T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Arch,Hardware]
series: [Use Arch]
series_weight: 4
featuredImagePreview: ""
summary: 本文所用显卡为 Intel Corporation UHD Graphics 620, 对于其他型号或其他品牌, 请参考 [AMDGPU](https://wiki.archlinux.org/title/AMDGPU), [AMDGPU PRO](https://wiki.archlinux.org/title/AMDGPU_PRO), [ATI](https://wiki.archlinux.org/title/ATI), [Intel Graphics](https://wiki.archlinux.org/title/Intel_graphics), [Nouveau](https://wiki.archlinux.org/title/Nouveau), [NVIDIA](https://wiki.archlinux.org/title/NVIDIA), [Vulkan](https://wiki.archlinux.org/title/Vulkan)
---

{{< admonition type=question title="显卡型号" open=true >}}
本文所用显卡为 Intel Corporation UHD Graphics 620, 对于其他型号或其他品牌, 请参考: 

[AMDGPU](https://wiki.archlinux.org/title/AMDGPU), [AMDGPU PRO](https://wiki.archlinux.org/title/AMDGPU_PRO), [ATI](https://wiki.archlinux.org/title/ATI), [Intel Graphics](https://wiki.archlinux.org/title/Intel_graphics), [Nouveau](https://wiki.archlinux.org/title/Nouveau), [NVIDIA](https://wiki.archlinux.org/title/NVIDIA), [Vulkan](https://wiki.archlinux.org/title/Vulkan)
{{< /admonition >}}

## 安装驱动

查看显卡类型: 

```bash
lspci | grep -e VGA -e 3D
```

参考 [Intel graphics](https://wiki.archlinux.org/title/Intel_graphics) 安装驱动: 

```bash
sudo pacman -S mesa vulkan-intel
```

32 位支持安不安装都可以: 

```bash
sudo pacman -S lib32-mesa lib32-vulkan-intel 
```

## 硬件视频加速

其他型号或其他品牌, 请参考 [Hardware video acceleration](https://wiki.archlinux.org/title/Hardware_video_acceleration)。


```bash
sudo pacman -S intel-media-driver libva-utils intel-gpu-tools
```

运行 `vainfo` 命令检查 VA-API 的设置。

播放视频时运行 `sudo intel_gpu_top` 命令, 如果 `Video` 行有占用, 说明正在使用硬件视频加速。

安装了上面的几个包之后, 本地视频播放成功使用硬件视频加速。

对于浏览器, 我看遍了论坛和 WiKi 并且折腾过几天, 最终放弃了让浏览器使用硬件视频加速。

这方面存在太多问题, Xorg+Firefox 能稳定硬件视频加速, 至于 Chromium, Chrome 和 Edge 等浏览器直接放弃即可。因为就算一时成功, 过段时间升级了浏览器或者驱动包, 或者因为其他原因, 硬件视频加速又会失效。

论坛上的相关帖子个个都有很多页, 有一个甚至有 30 多页, 时间跨度好几年。然而解决方案和出现的问题都是动态发展的, 没有一劳永逸甚至是稍微稳定的方法。

## 双显卡

对于 Intel 和 NVIDIA 双显卡, 参考 [NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus) 中的各种方案。

我在 Arch 上没打算用 NVIDIA, 于是参考 [Hybrid graphics#Using Udev Rules](https://wiki.archlinux.org/title/Hybrid_graphics#Using_udev_rules) 禁用了 nouveau: 

`sudo vim /etc/modprobe.d/blacklist-nouveau.conf` 并写入: 

```
blacklist nouveau
options nouveau modeset=0
```

