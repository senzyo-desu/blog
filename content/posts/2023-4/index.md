---
title: "Arch配置"
date: 2023-04-05T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Arch]
series: [Use Arch]
series_weight: 2
featuredImagePreview: ""
summary: Arch 安装完成之后的一些零散配置。
---

{{< admonition type=info title="明确要素" open=true >}}
物理机, 双系统, Arch, Windows, Btrfs, Xorg, SDDM, KDE。
{{< /admonition >}}

## 自定义终端

参考 [自定义终端](../2020-4/#自定义终端)。

## 同步时间

参考 [Systemd-timesyncd](https://wiki.archlinux.org/title/Systemd-timesyncd), `sudo vim /etc/systemd/timesyncd.conf` 来自定义 NTP 服务器: 

```
[Time]
NTP=0.cn.pool.ntp.org 1.cn.pool.ntp.org 2.cn.pool.ntp.org 3.cn.pool.ntp.org
FallbackNTP=0.arch.pool.ntp.org 1.arch.pool.ntp.org 2.arch.pool.ntp.org 3.arch.pool.ntp.org
```

启用时间同步: 

```bash
sudo timedatectl set-ntp true
```

重启服务: 

```bash
sudo systemctl restart systemd-timesyncd.service
```

检查配置: 

```bash
timedatectl show-timesync --all
```

检查同步状态: 

```bash
timedatectl timesync-status
```

## 使用中文locale

`sudo vim /etc/locale.gen` 写入以下内容: 

```bash
# en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

生成 locale 信息: 

```bash
sudo locale-gen
```

`sudo vim /etc/locale.conf` 写入以下内容: 

```bash
# LANG=en_US.UTF-8
LANG=zh_CN.UTF-8
```

使用 `locale` 命令查看区域信息。

{{< admonition type=question title="locale 还是不对, 且中英文混杂?" open=true >}}
KDE Plasma 也通过系统设置修改界面的语言, 如果按本文修改后还是原来的区域设置, 删除 `~/.config/plasma-localerc`。
{{< /admonition >}}
 
{{< admonition type=failure title="不推荐使用 Arch WiKi 的方法" open=true >}}
在 `/etc/locale.gen` 中使用 `en_US.UTF-8 UTF-8`, 然后 [在用户会话中覆盖系统区域设置](https://wiki.archlinuxcn.org/wiki/Locale#在用户会话中覆盖系统区域设置)。这样有可能会导致使用 `locale` 命令时被报错, 而且有些软件显示中文, 有些软件显示英文, 十分混乱。
{{< /admonition >}}

`locale.conf` 的变更会在下次登录时生效, 如果要立刻应用新的设置运行: 

```bash
unset LANG
source /etc/profile.d/locale.sh
```

## 休眠到硬盘

### 编辑mkinitcpio

```bash
sudo vim /etc/mkinitcpio.conf
```

找到 `HOOKS` 一行, 在括号中加入 `resume`, 并且保证 `resume` 一定要在 `udev` 之后, 比如这样: 

```
HOOKS=(base udev autodetect modconf kms keyboard keymap consolefont block filesystems resume fsck)
```

{{< admonition type=info title="提示" open=true >}}
每次编辑 `/etc/mkinitcpio.conf` 后都需要运行 `mkinitcpio -P` 命令重新生成 initramfs。
{{< /admonition >}}

### 编辑GRUB

运行 `sudo blkid` 获取分区的 UUID: 

```
/dev/sda2: UUID="1409b5bb-26f5-40a8-939f-251a38b8f1cb" UUID_SUB="02a23d46-8c29-4112-92c5-e3523f8b46ba" BLOCK_SIZE="4096" TYPE="btrfs" PARTUUID="4cb03e41-efa0-45ce-a102-827c01c4553a"
/dev/sda3: UUID="f381240f-4805-4032-9ebe-7b3a5fb53959" TYPE="swap" PARTUUID="ed0bf412-1817-4231-8b17-216095eeda89"
/dev/sda1: UUID="1008-AFCA" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="af60eb35-1357-4101-a464-ae685ae5fc91"
```

我的 SWAP 分区为 `/dev/sda3`, 对应的 UUID 为 `f381240f-4805-4032-9ebe-7b3a5fb53959`, 需要传递的内核参数为 `resume=UUID=f381240f-4805-4032-9ebe-7b3a5fb53959`。

```bash
sudo vim /etc/default/grub
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT` 一行, 将需要传递的内核参数填入引号中, 如: 

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet resume=UUID=f381240f-4805-4032-9ebe-7b3a5fb53959"
```

{{< admonition type=info title="提示" open=true >}}
每次编辑 `/etc/default/grub` 后都需要运行 `grub-mkconfig -o /boot/grub/grub.cfg` 命令重新生成 GRUB 主配置文件。
{{< /admonition >}}

## 缓解关机卡90秒的问题

Linux 系统在关机时, 偶尔会遇见卡住不动, 要等大约 90 秒才可以关机的情况。原因似乎是 systemd 以为有服务没有终止 (实际上没有), 于是等待服务自行停止或直到 90 秒后强行停止。

虽然不能完全解决这个问题, 但至少可以修改它等待的时间: 

`sudo vim /etc/systemd/system.conf`, 取消 `DefaultTimeoutStopSec=90s` 这一行的注释, 改用较短的时间, 比如 `DefaultTimeoutStopSec=5s`。

## 安装软件

### 常用的KDE软件

```bash
sudo pacman -S ark dolphin dolphin-plugins elisa gwenview kcalc kcolorchooser kdf konsole partitionmanager spectacle
```

| 包名             | 简介           |
| ---------------- | -------------- |
| ark              | 压缩软件       |
| dolphin          | 文件管理器     |
| dolphin-plugins  | 集成其他服务   |
| elisa            | 音乐播放器     |
| gwenview         | 图像查看器     |
| kcalc            | 计算器         |
| kcolorchooser    | 调色板、取色器 |
| kdf              | 磁盘占用查看器 |
| konsole          | 终端           |
| partitionmanager | 分区管理器     |
| spectacle        | 截图工具       |

### 其他软件

```bash
sudo pacman -S yay
```

```bash
yay -S 7-zip-full aria2 firefox git joplin-appimage kclock lunacy-bin ntfs-3g pot-translation telegram-desktop ttf-wps-fonts unrar visual-studio-code-bin vlc wps-office wps-office-mui-zh-cn
```

| 包名                   | 简介                                  |
| ---------------------- | ------------------------------------- |
| 7-zip-full             | 代替 p7zip 作为 ark 的依赖使其支持 7z |
| aria2                  | 下载器                                |
| firefox                | 浏览器                                |
| git                    | 版本控制                              |
| joplin-appimage        | 笔记软件                              |
| kclock                 | 多时钟、计时器、秒表和闹钟            |
| lunacy-bin             | 矢量图编辑器                          |
| ntfs-3g                | 挂载 NTFS 硬盘                        |
| pot-translation        | 翻译软件                              |
| telegram-desktop       | 聊天软件                              |
| ttf-wps-fonts          | WPS 符号字体                          |
| unrar                  | 作为 ark 的依赖使其支持 RAR           |
| visual-studio-code-bin | 文本编辑器                            |
| vlc                    | 音视频播放器                          |
| wps-office             | 办公软件                              |
| wps-office-mui-zh-cn   | WPS 中文语言包                        |

## 字体

Linux 字体配置属实是一大坑。

### 字体介绍

#### 更纱黑体

从 https://github.com/be5invis/Sarasa-Gothic/releases/latest 下载, 解压后的字体文件存放到 `/usr/share/fonts/Sarasa/`。

版本选择参考 [这里](https://zhuanlan.zhihu.com/p/627059922), 我使用 TTC, 比如 `Sarasa-TTC-1.0.0.7z`。

查看字体的 family:

```bash
fc-scan /usr/share/fonts/Sarasa/Sarasa-Regular.ttc --format='%{family}\n'
```

得到输出: 

```
Sarasa Gothic CL,更紗黑體 CL
Sarasa Gothic SC,更纱黑体 SC
Sarasa Gothic TC,更紗黑體 TC
Sarasa Gothic HC,更紗黑體 HC
Sarasa Gothic J,更紗ゴシック J
Sarasa Gothic K
Sarasa UI CL,更紗黑體 UI CL
Sarasa UI SC,更纱黑体 UI SC
Sarasa UI TC,更紗黑體 UI TC
Sarasa UI HC,更紗黑體 UI HC
Sarasa UI J,更紗ゴシック UI J
Sarasa UI K
Sarasa Mono CL,等距更紗黑體 CL
Sarasa Mono SC,等距更纱黑体 SC
Sarasa Mono TC,等距更紗黑體 TC
Sarasa Mono HC,等距更紗黑體 HC
Sarasa Mono J,更紗等幅ゴシック J
Sarasa Mono K
Sarasa Mono Slab CL,等距更紗黑體 Slab CL
Sarasa Mono Slab SC,等距更纱黑体 Slab SC
Sarasa Mono Slab TC,等距更紗黑體 Slab TC
Sarasa Mono Slab HC,等距更紗黑體 Slab HC
Sarasa Mono Slab J,更紗等幅ゴシック Slab J
Sarasa Mono Slab K
Sarasa Term CL
Sarasa Term SC
Sarasa Term TC
Sarasa Term HC
Sarasa Term J
Sarasa Term K
Sarasa Term Slab CL
Sarasa Term Slab SC
Sarasa Term Slab TC
Sarasa Term Slab HC
Sarasa Term Slab J
Sarasa Term Slab K
Sarasa Fixed CL
Sarasa Fixed SC
Sarasa Fixed TC
Sarasa Fixed HC
Sarasa Fixed J
Sarasa Fixed K
Sarasa Fixed Slab CL
Sarasa Fixed Slab SC
Sarasa Fixed Slab TC
Sarasa Fixed Slab HC
Sarasa Fixed Slab J
Sarasa Fixed Slab K
```

#### 思源宋体

安装: 

```bash
sudo pacman -S adobe-source-han-serif-otc-fonts
```

字体存放位置: `/usr/share/fonts/adobe-source-han-serif/`。

查看字体的 family:

```bash
fc-scan /usr/share/fonts/adobe-source-han-serif/SourceHanSerif-Regular.ttc --format='%{family}\n'
```

得到输出: 

```
Source Han Serif,源ノ明朝
Source Han Serif K,본명조
Source Han Serif SC,思源宋体
Source Han Serif TC,思源宋體
Source Han Serif HC,思源宋體 香港
```

#### Noto Fonts

安装: 

```bash
sudo pacman -S noto-fonts
```

字体存放位置: `/usr/share/fonts/noto/`。

我只用这三个: `NotoSans-Regular.ttf`、`NotoSerif-Regular.ttf` 和 `NotoSansMono-Regular.ttf`。

对应的 family 分别是 `Noto Sans`、`Noto Serif` 和 `Noto Sans Mono`。

#### Noto Color Emoji

安装: 

```bash
sudo pacman -S noto-fonts-emoji
```

字体路径: `/usr/share/fonts/noto/NotoColorEmoji.ttf`。

对应的 family 是 `Noto Color Emoji`。

#### Apple Color Emoji

{{< admonition type=info title="注意" open=true >}}
`ttf-apple-emoji` 与 `noto-fonts-emoji` 冲突, 安装一个必会卸载另一个。

当然可以手动将 ttf 文件复制保留到某个位置。不过多个 emoji 文件并存时, 系统不一定会使用哪个 emoji, 所以最好只保留一个 emoji 文件。
{{< /admonition >}}

安装: 

```bash
yay -S ttf-apple-emoji
```

字体路径: `/usr/share/fonts/TTF/AppleColorEmoji.ttf`。

对应的 family 是 `Apple Color Emoji`。

### KDE字体设置

我的情况是, 先调整 KDE 的“设置→显卡与显示器→显示器配置”, 1920x1080 缩放 125%, 于是下图中的字体 DPI 就变成了120。

调整 KDE 的“设置→外观→字体”: 

{{< image src="1.png" caption="KDE 字体设置" height="auto" width="60%">}}

保存设置后, `~/.config/fontconfig/fonts.conf` 的内容应该如下: 

```xml
<?xml version='1.0'?>
<fontconfig>
    <match target="font">
        <edit mode="assign" name="rgba">
            <const>rgb</const>
        </edit>
    </match>
    <match target="font">
        <edit mode="assign" name="hinting">
            <bool>false</bool>
        </edit>
    </match>
    <match target="font">
        <edit mode="assign" name="hintstyle">
            <const>hintnone</const>
        </edit>
    </match>
    <match target="font">
        <edit mode="assign" name="antialias">
            <bool>true</bool>
        </edit>
    </match>
</fontconfig>
```

更多 fontconfig 信息参考 [Fontconfig_configuration](https://wiki.archlinux.org/title/Font_configuration#Fontconfig_configuration)。

### 全局字体设置

{{< admonition type=info title="注意" open=true >}}
Linux 中, 用户的配置是会覆盖系统全局配置的, 也就是说, 如果 `~/.config/fontconfig/fonts.conf` 和 `/etc/fonts/local.conf` 中有相同的配置项, 前者会覆盖后者。
{{< /admonition >}}

所以我的 `~/.config/fontconfig/fonts.conf` 和 `/etc/fonts/local.conf` 中 **没有** 相同的配置项, 前者只保存来自 KDE 字体设置的配置信息, 后者保存自己高度自定义的配置信息。


- 参考 [Alias](https://wiki.archlinux.org/title/Font_configuration#Alias) 配置后备字体顺序; 
- 参考 [CJK,_but_other_Latin_fonts_are_preferred](https://wiki.archlinux.org/title/Font_configuration/Examples#CJK,_but_other_Latin_fonts_are_preferred) 配置 CJK; 
- 参考 [Whitelisting_and_blacklisting_fonts](https://wiki.archlinux.org/title/Font_configuration#Whitelisting_and_blacklisting_fonts) 配置字体黑白名单; 
- 参考 [Emoji在输入法候选框中不显示](../2023-5/#emoji在输入法候选框中不显示) 为 Emoji 启用微调, 确保 Emoji 能在输入法候选框中正常显示; 
- 参考 [fontconfig 几个常见的坑](https://marguerite.su/posts/fontconfig_几个常见的坑/) 设置 Firefox 不使用自带的 `Twemoji Mozilla` 而使用自定义的 `Noto Color Emoji` 或者 `Apple Color Emoji`。

`sudo vim /etc/fonts/local.conf`:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
    <!-- Fallback fonts preference order -->

    <!-- Default serif font -->
    <alias binding="strong">
        <family>serif</family>
        <prefer>
            <family>Source Han Serif SC</family>
            <family>Noto Serif</family>
            <!-- <family>Apple Color Emoji</family> -->
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <!-- Default sans-serif font -->
    <alias binding="strong">
        <family>sans-serif</family>
        <prefer>
            <family>Sarasa UI SC</family>
            <family>Noto Sans</family>
            <!-- <family>Apple Color Emoji</family> -->
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <!-- Default monospace font -->
    <alias binding="strong">
        <family>monospace</family>
        <prefer>
            <family>Sarasa Mono SC</family>
            <family>Noto Sans Mono</family>
            <!-- <family>Apple Color Emoji</family> -->
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <!-- Default system-ui font -->
    <alias binding="strong">
        <family>system-ui</family>
        <prefer>
            <family>Sarasa UI SC</family>
            <family>Noto Sans</family>
            <!-- <family>Apple Color Emoji</family> -->
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>

    <!-- Serif CJK -->

    <!-- Default serif when the "lang" attribute is not given -->
    <!-- You can change this font to the language variant you want -->
    <match target="pattern">
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif SC</string>
        </edit>
    </match>

    <!-- Japanese -->
    <!-- "lang=ja" or "lang=ja-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ja</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif</string>
        </edit>
    </match>

    <!-- Korean -->
    <!-- "lang=ko" or "lang=ko-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ko</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif K</string>
        </edit>
    </match>

    <!-- Chinese -->
    <!-- "lang=zh" or "lang=zh-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hans" or "lang=zh-hans-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hans</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant" or "lang=zh-hant-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant-hk" or "lang=zh-hant-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant-hk</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif HC</string>
        </edit>
    </match>
    <!-- Compatible -->
    <!-- "lang=zh-cn" or "lang=zh-cn-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-cn</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif SC</string>
        </edit>
    </match>
    <!-- "lang=zh-tw" or "lang=zh-tw-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-tw</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hk" or "lang=zh-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hk</string>
        </test>
        <test name="family">
            <string>serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Source Han Serif HC</string>
        </edit>
    </match>

    <!-- Sans CJK -->

    <!-- Default sans-serif when the "lang" attribute is not given -->
    <!-- You can change this font to the language variant you want -->
    <match target="pattern">
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>

    <!-- Japanese -->
    <!-- "lang=ja" or "lang=ja-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ja</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI J</string>
        </edit>
    </match>

    <!-- Korean -->
    <!-- "lang=ko" or "lang=ko-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ko</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI K</string>
        </edit>
    </match>

    <!-- Chinese -->
    <!-- "lang=zh" or "lang=zh-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hans" or "lang=zh-hans-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hans</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant" or "lang=zh-hant-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant-hk" or "lang=zh-hant-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant-hk</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI HC</string>
        </edit>
    </match>
    <!-- Compatible -->
    <!-- "lang=zh-cn" or "lang=zh-cn-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-cn</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-tw" or "lang=zh-tw-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-tw</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hk" or "lang=zh-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hk</string>
        </test>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI HC</string>
        </edit>
    </match>

    <!-- Mono CJK -->

    <!-- Default monospace when the "lang" attribute is not given -->
    <!-- You can change this font to the language variant you want -->
    <match target="pattern">
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono SC</string>
        </edit>
    </match>

    <!-- Japanese -->
    <!-- "lang=ja" or "lang=ja-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ja</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono J</string>
        </edit>
    </match>

    <!-- Korean -->
    <!-- "lang=ko" or "lang=ko-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ko</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono K</string>
        </edit>
    </match>

    <!-- Chinese -->
    <!-- "lang=zh" or "lang=zh-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hans" or "lang=zh-hans-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hans</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant" or "lang=zh-hant-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant-hk" or "lang=zh-hant-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant-hk</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono HC</string>
        </edit>
    </match>
    <!-- Compatible -->
    <!-- "lang=zh-cn" or "lang=zh-cn-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-cn</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono SC</string>
        </edit>
    </match>
    <!-- "lang=zh-tw" or "lang=zh-tw-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-tw</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hk" or "lang=zh-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hk</string>
        </test>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa Mono HC</string>
        </edit>
    </match>

    <!-- System UI CJK -->

    <!-- Default system-ui when the "lang" attribute is not given -->
    <!-- You can change this font to the language variant you want -->
    <match target="pattern">
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>

    <!-- Japanese -->
    <!-- "lang=ja" or "lang=ja-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ja</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI J</string>
        </edit>
    </match>

    <!-- Korean -->
    <!-- "lang=ko" or "lang=ko-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>ko</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI K</string>
        </edit>
    </match>

    <!-- Chinese -->
    <!-- "lang=zh" or "lang=zh-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hans" or "lang=zh-hans-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hans</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant" or "lang=zh-hant-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hant-hk" or "lang=zh-hant-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hant-hk</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI HC</string>
        </edit>
    </match>
    <!-- Compatible -->
    <!-- "lang=zh-cn" or "lang=zh-cn-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-cn</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI SC</string>
        </edit>
    </match>
    <!-- "lang=zh-tw" or "lang=zh-tw-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-tw</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI TC</string>
        </edit>
    </match>
    <!-- "lang=zh-hk" or "lang=zh-hk-*" -->
    <match target="pattern">
        <test name="lang" compare="contains">
            <string>zh-hk</string>
        </test>
        <test name="family">
            <string>system-ui</string>
        </test>
        <edit name="family" mode="append" binding="strong">
            <string>Sarasa UI HC</string>
        </edit>
    </match>

    <!-- Whitelisting and blacklisting fonts -->
    <selectfont>
        <rejectfont>
            <pattern><patelt name="family"><string>Sarasa Gothic CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Gothic SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Gothic TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Gothic HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Gothic J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Gothic K</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa UI CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Mono Slab K</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term K</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Term Slab K</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed K</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab CL</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab SC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab TC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab HC</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab J</string></patelt></pattern>
            <pattern><patelt name="family"><string>Sarasa Fixed Slab K</string></patelt></pattern>
            <glob>/usr/share/fonts/adobe-source-code-pro/*</glob>
            <glob>/usr/share/fonts/cantarell/*</glob>
            <glob>/usr/share/fonts/liberation/*</glob>
            <!-- <glob>/usr/share/fonts/noto/*</glob> -->
            <glob>/usr/share/fonts/TTF/*</glob>
        </rejectfont>
        <acceptfont>
            <!-- <glob>/usr/share/fonts/noto/NotoSans-Regular.ttf</glob> -->
            <!-- <glob>/usr/share/fonts/noto/NotoSerif-Regular.ttf</glob> -->
            <!-- <glob>/usr/share/fonts/noto/NotoSansMono-Regular.ttf</glob> -->
            <!-- <glob>/usr/share/fonts/noto/NotoColorEmoji.ttf</glob> -->
            <!-- <glob>/usr/share/fonts/TTF/AppleColorEmoji.ttf</glob> -->
        </acceptfont>
    </selectfont>

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

    <!-- Ensure Firefox uses custom emoji instead of built-in Twemoji Mozilla -->
    <match>
        <test name="family" compare="eq">
            <string>Twemoji Mozilla</string>
        </test>
        <test name="prgname" compare="eq">
            <string>firefox</string>
        </test>
        <edit name="family" mode="prepend">
            <!-- <string>Apple Color Emoji</string> -->
            <string>Noto Color Emoji</string>
        </edit>
    </match>
</fontconfig>
```

清除字体缓存: 

```bash
fc-cache -fv
```

重启电脑或者重启 `sddm` 服务立即生效: 

```bash
sudo systemctl restart sddm
```

## 输入法

更多信息参考 [Rime配置](../2023-2/)。

## 默认编辑器

编辑 [环境变量配置文件](../2021-6/), 添加以下行, 将 vim 设置为默认编辑器: 

```
export EDITOR="/usr/bin/vim"
```

## 音频

{{< admonition type=warning title="注意" open=true >}}
配置音频需要使用普通用户身份。
{{< /admonition >}}

推荐使用 [PipeWire](https://wiki.archlinux.org/title/PipeWire) 代替 KDE 默认的 [PulseAudio](https://wiki.archlinux.org/title/PulseAudio)。

安装 `pipewire-pulse` 替代 `pulseaudio` 和 `pulseaudio-bluetooth`: 

```bash
sudo pacman -S pipewire-pulse
```

重启电脑或者运行以下命令: 

```bash
systemctl --user daemon-reload
systemctl stop --user pulseaudio.service
systemctl start --user pipewire-pulse.service
```

使用 `pactl info`, 输出中应有: 

```
Server Name: PulseAudio (on PipeWire x.y.z)
```

更多信息参考 [Sound system](https://wiki.archlinux.org/title/Sound_system)。

## TTS

使用 Firefox 的时候, 最上面会提示没有安装 Speech Dispatcher。安装: 

```bash
sudo pacman -S speech-dispatcher
```

提示从 `festival` 或 `espeak-ng` 中选一个作为依赖, 推荐后者。

这里只是安装上以防万一用到, 实际上 `espeak-ng` 的效果并不好, 如果追求更好的 TTS 效果, 推荐阅读 [TTS Awesome](https://wener.me/notes/ai/tts/awesome)。

## 蓝牙

安装: 

```bash
sudo pacman -S bluez bluez-utils bluedevil
```

设置蓝牙服务开机自启并立刻启动: 

```bash
sudo systemctl enable --now bluetooth.service
```

更多信息参考 [Bluetooth](https://wiki.archlinux.org/title/Bluetooth)。

### 蓝牙耳机

使用蓝牙耳机播放音频时, 检测当前使用编码: 

```bash
pactl list | grep codec
```

更多信息参考 [Bluetooth headset](https://wiki.archlinux.org/title/Bluetooth_headset)。

## 启动时打开数字锁定键

1. 先设置 KDE Plasma: 

    “系统设置→输入设备→键盘→Plasma 启动时 NumLock 状态”: 开启。

2. 再设置 SDDM: 

    `sudo vim /etc/sddm.conf`, 在配置文件中的 `[General]` 部分添加 `Numlock=on`。

更多信息参考 [Activating numlock on bootup](https://wiki.archlinux.org/title/Activating_numlock_on_bootup)。

## 固态硬盘TRIM

如果参考前文, 在 [挂载子卷](../2023-3/#挂载子卷) 时使用了 `discard=async` 选项, 这部分跳过即可。

更多信息参考 [Solid state drive#TRIM](https://wiki.archlinux.org/title/Solid_state_drive#TRIM)。

### 检查支持

```bash
sudo pacman -S hdparm
```

检查 `/dev/sda` 是否支持 TRIM: 

```bash
hdparm -I /dev/sda | grep TRIM
```

如果支持应当得到输出: 

```
*    Data Set Management TRIM supported (limit 8 blocks)
```

### 启用fstrim

```bash
sudo pacman -S util-linux
```

启用 `fstrim.timer` 计时器会在每周激活服务, 在所有已挂载的支持 discard 操作的文件系统上执行 [fstrim](https://man.archlinux.org/man/fstrim.8): 

```bash
sudo systemctl start fstrim.timer
```

## 开机速度

参考 [Linux开机速度慢](../2020-6/#linux开机速度慢)。

## 多内核

根据自己的需要选择是否使用多内核。

更多信息参考 [Kernel#Officially_supported_kernels](https://wiki.archlinux.org/title/Kernel#Officially_supported_kernels)。

Linux Default Kernel:

```bash
sudo pacman -S linux linux-headers
```

Linux Hardened Kernel:

```bash
sudo pacman -S linux-hardened linux-hardened-headers 
```

Linux LTS Kernel:

```bash
sudo pacman -S linux-lts linux-lts-headers
```

Linux Zen Kernel:

```bash
sudo pacman -S linux-zen linux-zen-headers
```

## 自定义GRUB

参考 [自定义GRUB](../2020-12/)。

## 触控板手势

```bash
sudo gpasswd -a $USER input
```

重启系统。然后安装 libinput-gestures:

```bash
yay -S libinput-gestures
```

复制配置:

```bash
cp /etc/libinput-gestures.conf ~/.config/libinput-gestures.conf
```

参考 [原配置文件](https://github.com/bulletmark/libinput-gestures/blob/master/libinput-gestures.conf) 和 [README](https://github.com/bulletmark/libinput-gestures/blob/master/README.md), 编辑配置文件 `~/.config/libinput-gestures.conf` 来自定义触控板手势, 记得提前在 Gnome 或 KDE 的设置中配置相应的快捷键。

{{< admonition type=warning title="注意" open=true >}}
如果触控板未使用 `自然滚动`, 则需要交换下面参考配置中 `motion` 处的 `left/right` 和 `up/down`。
`left_up` 等对角线方向不用替换, 它们不受 `自然滚动` 影响。
{{< /admonition >}}

```
# 文档: https://github.com/bulletmark/libinput-gestures/blob/master/README.md
# 重启: libinput-gestures-setup restart
# 格式: action motion [finger_count] command
###########################################################
# 滑动手势
#############################
# 3 指手势
# 快速铺放窗口到左上方
gesture swipe left_up 3 xdotool key super+KP_Home
# 快速铺放窗口到上方
gesture swipe up 3 xdotool key super+Up
# 快速铺放窗口到右上方
gesture swipe right_up 3 xdotool key super+KP_Page_Up
# 快速铺放窗口到左侧
gesture swipe left 3 xdotool key super+Left
# 打开 Konsole
gesture hold on 3 konsole
# 快速铺放窗口到右侧
gesture swipe right 3 xdotool key super+Right
# 快速铺放窗口到左下方
gesture swipe left_down 3 xdotool key super+KP_End
# 快速铺放窗口到下方
gesture swipe down 3 xdotool key super+Down
# 快速铺放窗口到右下方
gesture swipe right_down 3 xdotool key super+KP_Page_Down
#############################
# 4 指手势
# 窗口移动到上一桌面
gesture swipe left_up 4 xdotool key super+control+shift+Tab
# 显示桌面总览
gesture swipe up 4 xdotool key super+w
# 窗口移动到下一桌面
gesture swipe right_up 4 xdotool key super+control+Tab
# 切换到下一桌面
gesture swipe left 4 xdotool key super+Tab
# 最大化窗口
gesture hold on 4 xdotool key super+x
# 切换到上一个桌面
gesture swipe right 4 xdotool key super+shift+Tab
# 打开 Dolphin
gesture swipe left_down 4 dolphin
# 最小化窗口
gesture swipe down 4 xdotool key super+n
# 暂时显示桌面
gesture swipe right_down 4 xdotool key super+d
###########################################################
# 捏手势
# 截屏矩形区域
gesture pinch in xdotool key super+shift+Print
```

### 操作命令

要立即 (重新) 启动应用程序, 并使其能够在登录时自动启动: 

```bash
libinput-gestures-setup stop desktop autostart start
```

登录时自动启动应用程序: 

```bash
libinput-gestures-setup autostart
```

禁止登录时自动启动应用程序: 

```bash
libinput-gestures-setup autostop
```

立即启动应用程序: 

```bash
libinput-gestures-setup start
```

立即停止应用程序: 

```bash
libinput-gestures-setup stop
```

重新启动应用程序, 例如重新加载配置文件: 

```bash
libinput-gestures-setup restart
```

检查应用程序的状态: 

```bash
libinput-gestures-setup status
```

## 电源管理

安装 `power-profiles-daemon`: 

```bash
sudo pacman -S power-profiles-daemon
```

设置 `power-profiles-daemon` 在登录时自动启动: 

```bash
sudo systemctl enable power-profiles-daemon.service
```

重启, 然后在系统托盘的 `电量和电池` 项目中使用电源管理方案。

--------------------

{{< admonition type=abstract title="推荐阅读" open=true >}}
1. [General recommendations](https://wiki.archlinux.org/title/General_recommendations)
2. [System maintenance](https://wiki.archlinux.org/title/System_maintenance)
3. [Environment variables](https://wiki.archlinux.org/title/Environment_variables)
4. [List of applications](https://wiki.archlinux.org/title/List_of_applications)
5. [Pacman](https://wiki.archlinux.org/title/Pacman)
6. [Pacman/Tips_and_tricks](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks)
{{< /admonition >}}
