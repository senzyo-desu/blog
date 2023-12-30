---
title: "Arch使用问题"
date: 2023-04-06T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Arch,Help]
series: [Use Arch]
series_weight: 3
featuredImagePreview: ""
summary: 使用 Arch 时遇到的问题及其解决方法。
---

## 从睡眠唤醒无声音输出

看了好几篇 WiKi 和论坛帖子, 尝试了一些方法并无效果。

检查 `lspci -vv` 和 `lsmod` 的输出, 在睡眠前后并无变化, 说明不是驱动的问题: 

最后得到 [Arch 中文群组](https://t.me/archlinuxcn_group) 群友的解惑, 原因是 EAPD (External Amplifier Power Down, 外部放大器断电): 电脑睡眠的时候关闭音频放大器, 如果配置不正确, 唤醒之后无法重新激活放大器, 就会没有声音。这是厂商不按标准来设计音频电路导致的问题。

如果在 `/etc/modprobe.d/modprobe.conf` 中设置 `model=auto` 无效: 

```
options snd-hda-intel model=auto
```

那就要从内核中查找相应的 model, 比如 Realtek 声卡需要查找 https://github.com/torvalds/linux/blob/v6.2/sound/pci/hda/patch_realtek.c 

安装 `alsa-utils` 包, 使用 `aplay -l` 列出所有声卡和数字音频设备, 得知当前声卡为 `ALC256`: 

```
**** List of PLAYBACK Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC256 Analog [ALC256 Analog]
Subdevices: 0/1
Subdevice #0: subdevice #0
```

在 [patch_realtek.c](https://github.com/torvalds/linux/blob/v6.2/sound/pci/hda/patch_realtek.c) 中查找 `ALC256_FIXUP` 和 `ALC256_FIXUP_SET_COEF_DEFAULTS`, 终于找到这样一行: 

```
SND_PCI_QUIRK(0x1d05, 0x1132, "TongFang PHxTxX1", ALC256_FIXUP_SET_COEF_DEFAULTS),
```

`0x1d05, 0x1132`, 即 `1d05:1132`, 在 `/etc/modprobe.d/modprobe.conf` 中写入: 

```
options snd-hda-intel model=1d05:1132
```

睡眠再唤醒后声音输出正常, 成功解决问题。

## 蓝牙与D-Bus实验性功能

一个终端实时监测蓝牙日志: 

```bash
journalctl -f | grep -i bluetooth
```

另一个终端重启蓝牙服务: 

```bash
sudo systemctl restart bluetooth.service
```

这时日志输出中有以下报错: 

```
bluetoothd[17935]: profiles/audio/vcp.c:vcp_init() D-Bus experimental not enabled
bluetoothd[17935]: src/plugin.c:plugin_init() Failed to init vcp plugin
bluetoothd[17935]: profiles/audio/mcp.c:mcp_init() D-Bus experimental not enabled
bluetoothd[17935]: src/plugin.c:plugin_init() Failed to init mcp plugin
bluetoothd[17935]: profiles/audio/bap.c:bap_init() D-Bus experimental not enabled
bluetoothd[17935]: src/plugin.c:plugin_init() Failed to init bap plugin
```

报告当前耳机电量、让 `journalctl` 输出的蓝牙日志中不再有以上报错, 都需要开启 D-Bus 的 [实验性功能](https://wiki.archlinux.org/title/Bluetooth#Enabling_experimental_features): 

`sudo vim /etc/bluetooth/main.conf`, 将 `Experimental` 的值改为 `true`: 

```
# Enables D-Bus experimental interfaces
# Possible values: true or false
Experimental = true
```

重启蓝牙服务: 

```bash
sudo systemctl restart bluetooth.service
```

可以发现日志中不再报告上面的错误。

## 密码试错锁定

`/etc/security/faillock.conf` 中有几个选项: 

```
# deny = 3
# fail_interval = 900
# unlock_time = 600
```

意思是 900 秒(15 分钟)内发生 3 次密码输入错误, 锁定该账户 600 秒(10 分钟), 即使再输入正确的密码也不行。

建议根据自身需求来更改。

## 模块固件缺失的警告

当内核更新后, 镜像initramfs被重新构建时, 你可能得到以下警告:

```
==> WARNING: Possibly missing firmware for module: wd719x
==> WARNING: Possibly missing firmware for module: aic94xx
==> WARNING: Possibly missing firmware for module: xhci_pci
```

实际上一般用户根本用不到它们, 所以完全不用处理, 参考 [Mkinitcpio#Possibly_missing_firmware_for_module_XXXX](https://wiki.archlinux.org/title/Mkinitcpio#Possibly_missing_firmware_for_module_XXXX)。

## Emoji在输入法候选框中不显示

当我在 KDE 的“设置→外观→字体”中设置微调 (`hinting`) 为 `无` (`hintnone`) 和 `轻微` (`hintslight`) 时, 字体显示正常; 设置微调为 `适中` (`hintmedium`) 和 `完全` (`hintfull`) 时, 有些软件中的字体会变形。

所以我没有使用微调, 于是就遇上了著名的 [Color emojis are not displayed](https://bugs.freedesktop.org/show_bug.cgi?id=104542) 的问题。

解决方法是确保 Emoji 文件启用微调。

可以全部字体文件启用轻微程度的微调, 比如调整 KDE 的“设置→外观→字体”: “微调→轻微”。

不过更推荐在配置文件中为 Emoji 文件单独启用微调, 这样就算以后在 KDE 字体设置中更改了微调设置也不影响 Emoji 的显示。

`sudo vim /etc/fonts/local.conf` 或者 `vim ~/.config/fontconfig/fonts.conf`, 添加以下片段: 

```xml
    <!-- Ensure input method displays emoji -->
    <match target="font">
        <test name="family" compare="contains">
            <string>Emoji</string>
        </test>
        <edit name="hinting" mode="assign">
            <bool>true</bool>
        </edit>
        <edit name="hintstyle" mode="assign">
            <const>hintslight</const>
        </edit>
        <edit name="embeddedbitmap" mode="assign">
            <bool>true</bool>
        </edit>
    </match>
```

