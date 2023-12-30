---
title: "Arch安装"
date: 2023-04-04T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Arch]
series: [Use Arch]
series_weight: 1
featuredImagePreview: ""
summary: 物理机, 双系统, Arch, Windows, Btrfs, Xorg, SDDM, KDE。
---

{{< admonition type=info title="明确要素" open=true >}}
物理机, 双系统, Arch, Windows, Btrfs, Xorg, SDDM, KDE。
{{< /admonition >}}

## 下载ISO

从 [Arch Linux Downloads](https://archlinux.org/download/) 或 [校园网联合镜像站](https://mirrors.cernet.edu.cn/list/archlinux) 下载, 推荐 BitTorrent 方式。

## 安装介质

用 [Ventoy](https://github.com/ventoy/Ventoy/releases/latest) 制作 U 盘启动盘。

## 启动到Live环境

按 `F2` 或 `F12` 或 `Del` 键进入 UEFI 模式, 关闭安全启动, 将 U 盘设置为首位加载。

{{< admonition type=warning title="注意" open=true >}}
Arch Linux 安装镜像不支持 UEFI 安全启动 (Secure Boot) 功能。如果要引导安装介质, 需要禁用安全启动。如果需要, 可在完成安装后重新配置。
{{< /admonition >}}

现在电脑一般都是以 UEFI 模式引导且使用 x64 UEFI, 所以懒得 [验证引导模式](https://wiki.archlinux.org/title/Installation_guide#Verify_the_boot_mode)。

## 连接网络

使用网线最为方便, 不用额外设置; 不推荐手机 USB 共享网络; 下文介绍使用无线网络的情况。

### 确保硬件正常

检查网络接口的打开状态: `ip link`, 输出类似: 

```
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state DOWN mode DORMANT group default qlen 1000
    link/ether 12:34:56:78:9a:bc brd ff:ff:ff:ff:ff:ff
```

`<BROADCAST,MULTICAST,UP,LOWER_UP>` 中的 `UP` 表示接口已经打开, 不要和后面的 `state DOWN` 混淆。

如果 `wlan0` 确实未打开, 运行: 

```bash
ip link set wlan0 up
```

如果出现错误 `RTNETLINK answers: Operation not possible due to RF-kill`, 说明网卡未被 [rfkill](https://wiki.archlinux.org/title/rfkill) 禁用了, 使用 `rfkill` 命令来检查, 正常输出类似: 

```
ID TYPE      DEVICE      SOFT      HARD
 0 bluetooth hci0   unblocked unblocked
 1 wlan      phy0   unblocked unblocked
```

如果 `wlan` 处于硬件屏蔽 (hard-blocked) 状态, 使用硬件按钮或开关来开启它; 如果 `wlan` 并没有被硬件屏蔽但处于软件屏蔽 (soft-blocked) 状态, 运行: 

```bash
rfkill unblock wlan
```

### 连接无线网络

参考 [Iwd#iwctl](https://wiki.archlinux.org/title/Iwd#iwctl), 输入 `iwctl` 命令进入 `iwd` 模式。

列出所有 Wi-Fi 设备: 

```bash
device list
```

如果设备或其相应的适配器已关闭, 将其打开: 

```bash
device <device> set-property Powered on
adapter <adapter> set-property Powered on
```

扫描网络 (这个命令不会输出任何内容) : 

```bash
station <device> scan
```

列出所有可用的网络: 

```bash
station <device> get-networks
```

连接到一个网络: 

```bash
station <device> connect <SSID>
```

如果需要网络密码, 将会提示用户输入。

最后输入 `exit` 或按 `Ctrl+D` 退出 `iwd` 模式。

测试网络: 

```bash
ping www.baidu.com
```

## TTY字体

默认的字体实在是没法看, 查看所有适用于 TTY 的字体:

```bash
ls /usr/share/kbd/consolefonts
```

临时设置一个字体: 

```bash
setfont LatGrkCyr-12x22
```

或者编辑 `/etc/vconsole.conf`, 设置默认字体: 

```
FONT=LatGrkCyr-12x22
```

这个字体虽然也不好看, 但又不是不能用, 到处都在推荐的 `terminus-font` 字体也没好看哪里去。

{{< admonition type=note title="安装 terminus-font" open=false >}}
```bash
pacman -S terminus-font
```

然后按照上面的流程选择字体。

关于 `terminus-font` 字体名称中的 `g*` 、`*n` 之类的参数, 参阅 `/usr/share/kbd/consolefonts/README.Lat2-Terminus16`。
{{< /admonition >}}

## 确保系统时间准确

```bash
timedatectl status
``` 

连接网络后, 时间会自动同步, 更多信息参阅 [timedatectl](https://man.archlinux.org/man/timedatectl.1)。

## 硬盘分区

### 双系统EFI分区问题

双系统的 EFI 分区是个麻烦, 原因: 

1. Windows 的 EFI 分区太小了, 区区 100M 根本不满足 Windows 和 Arch 两个系统使用, 将直接导致安装 Arch 失败。
2. 一块硬盘上只能有一个 EFI 分区。

不能一味地遵循 Arch WiKi 和各路文档的做法, 直接挂载已有的 EFI 分区, 那只适用于只有一块硬盘且已有的 EFI 分区足够大的情况。

#### 一块硬盘

如果只有一块硬盘且已经安装了 Windows, 要么扩容 Windows 的 EFI 分区 (但很容易出问题), 要么干脆重装: 

1. 提前将 Windows 数据备份。
2. 在安装 Arch 的 Live 环境中, 先删除目标硬盘的所有分区, 再使用 `cfdisk` 命令对目标硬盘进行分区 (比如 `cfdisk /dev/sda`), 目标是创建一个大的 EFI 分区。
3. 使用 `mkfs.fat` 命令格式化 EFI 分区 (比如 `mkfs.fat -F 32 /dev/sda1`)。
4. 退出 Live 环境。
5. 先安装 Windows, Windows 会自动使用已有 EFI 分区。
6. 再安装 Arch, 安装 Arch 时手动挂载这个 EFI 分区。

双系统安装在一块硬盘上的缺点: 

1. EFI 分区不够大。
2. 硬盘故障牵连两个系统。
3. 重装 Windows 系统会造成 Linux 启动文件丢失。

#### 两块硬盘

如果有两块硬盘, 那么最佳实践是将两个系统分别安装在两块不同的硬盘中, 两个系统各自使用自己的 EFI 分区。

要想双启动, 就将 Windows 的启动文件复制到 Arch 的 EFI 分区中, 在 UEFI 模式中将 GRUB 调整为首位加载, 每次开机就可以选择启动 Arch 还是 Windows 了。

本文就采用这种方式。

### 创建分区

使用 `lsblk -f` 或 `fdisk -l` 查看所有硬盘分区信息。

使用 `cfdisk` 命令对目标硬盘进行分区 (比如 `cfdisk /dev/sda`)。

`cfdisk` 的界面足够简洁明了, 先 `New` 来新建分区, 再 `Type` 选择分区类型, 之后 `Write` 将分区信息写入分区表, 最后 `Quit` 退出。

EFI 分区建议分配 1G, 因为说不定后续有安装多内核的需求、同硬盘安装其他 Linux 的需求, 反正现在的硬盘容量都很大, 不差这点。

`fdisk -l` 查看分区情况: 

```
设备            起点       末尾       扇区    大小  类型
/dev/sda1       2048    2099199    2097152      1G  EFI 系统
/dev/sda2    2099200  211814399  209715200    100G  Linux 文件系统
/dev/sda3  211814400  245368831   33554432     16G  Linux swap

设备            起点       末尾       扇区    大小  类型
/dev/sdb1       2048     206847     204800    100M  EFI 系统
/dev/sdb2     206848     239615      32768     16M  Microsoft 保留
/dev/sdb3     239616  498708479  498468864  237.7G  Microsoft 基本数据
/dev/sdb4  498708480  500115455    1406976    687M  Windows 恢复环境
```

### 格式化分区

格式化 EFI 分区:

```bash
mkfs.fat -F 32 /dev/sda1
```

格式化根分区: 

```bash
mkfs.btrfs /dev/sda2
```

初始化交换空间分区: 

```bash
mkswap /dev/sda3
```

{{< admonition type=warning title="警告" open=true >}}
只有在分区步骤中创建 EFI 系统分区时才需要格式化。如果这个磁盘上已经有一个 EFI 系统分区了, 将它重新格式化会破坏其他已安装操作系统的引导加载程序。
{{< /admonition >}}

### 挂载分区

{{< admonition type=warning title="注意" open=true >}}
一定要按照顺序挂载。
{{< /admonition >}}

#### 挂载根分区

##### 创建子卷

```bash
mount /dev/sda2 /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
umount /dev/sda2
```

##### 挂载子卷

```bash
mount /dev/sda2 /mnt -o subvol=@,noatime,discard=async
mount --mkdir /dev/sda2 /mnt/home -o subvol=@home,noatime,discard=async
```

`subvol` 选项: 

1. 第一个选项用来指定挂载的子卷。
2. `noatime` 选项可以降低数据读取和写入的访问时间。
3. `discard=async` 选项可以在闲时释放磁盘中未使用的区块, 也就是 TRIM。也可以不添加这个选项, 而是在系统安装完成后启用 `fstrim.timer` 服务从而定时执行 TRIM, 参考 [固态硬盘TRIM](../2023-4/#固态硬盘trim)。
4. `compress` 选项参考 [Btrfs#Compression](https://wiki.archlinux.org/title/btrfs#Compression), 这里就不折腾了, 我觉得没必要。

在系统安装完成后也可以编辑 `/etc/fstab` 文件修改挂载选项。

#### 挂载EFI分区

```bash
mount --mkdir /dev/sda1 /mnt/boot
```

这时可以挂载 Windows 的 EFI 分区, 以便之后使用 `os-prober` 探测系统: 

```bash
mount --mkdir /dev/sdb1 /mnt/windowsboot
```

#### 挂载交换空间分区

```bash
swapon /dev/sda3
```

## 筛选镜像

使用 `reflector` 筛选镜像源保存到文件中: 

```bash
reflector --download-timeout 300 --threads 8 --verbose --country China --fastest 5 --latest 5 --protocol https --completion-percent 99 --isos --ipv6 --save /etc/pacman.d/mirrorlist
```

网络原因可能导致这一工作没有很好的完成, 一定要 `cat /etc/pacman.d/mirrorlist` 检查镜像源, 确保镜像地址正确地被写入文件中。

或者手动输入镜像地址:

```bash
vim /etc/pacman.d/mirrorlist
```

```
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch
```

## Pacman配置

参考 [Pacman](https://wiki.archlinux.org/title/Pacman) 和 [Pacman/Tips_and_tricks](https://wiki.archlinux.org/title/Pacman/Tips_and_tricks)。

`vim /etc/pacman.conf`: 

- 启用并行下载: 设置 `Misc options` 选项下的 `ParallelDownloads = 5`, 将会同时下载 5 个软件包, 如果未设置此选项, 软件包将会被依次下载。
- 启用着色: 将 `Misc options` 选项下的 `Color` 取消注释。

## 安装必需的软件包

```bash
pacstrap -K /mnt base base-devel linux linux-headers linux-firmware os-prober intel-ucode btrfs-progs networkmanager vim
```

微码: AMD 处理器安装 `amd-ucode`, Intel 处理器安装 `intel-ucode`。

网络工具: 只安装 `networkmanager` 即可。如果安装 `iwd`, 由于它与 `networkmanager` 冲突, 最后不免要手动关闭它。

## Fstab

生成 `fstab` 文件: 

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

检查生成的 `/mnt/etc/fstab` 文件是否正确: 

```bash
cat /mnt/etc/fstab
```

## Chroot

chroot 到新安装的系统: 

```bash
arch-chroot /mnt
```

## 网络配置

### 自定义主机名称

```bash
vim /etc/hostname
```

### 连接网络

这一步只有 Wi-Fi才需要配置; 如果插着网线, 那网络是一直连通的。

设置 `networkmanager` 开机自启并立刻启动: 

```bash
systemctl enable --now NetworkManager
```

参考 [NetworkManager#nmcli examples](https://wiki.archlinux.org/title/NetworkManager#nmcli_examples) 来使用 Wi-Fi。

显示附近的 Wi-Fi 网络: 

```bash
nmcli device wifi list
```

连接到 Wi-Fi 网络: 

```bash
nmcli device wifi connect <SSID_or_BSSID> password <password>
```

## 设置时间

设置时区: 

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

硬件时间为 UTC 时间的前提下, 运行以下命令来生成 `/etc/adjtime`: 

```bash
hwclock --systohc
```

因为我只用 `UTC+8` 的 `Asia/Shanghai` 这一个时区, 而且中国也不用夏令时, 所以我选择使用 `localtime` 而不是 `UTC`。

将硬件时间改为 localtime: 

```bash
timedatectl set-local-rtc 1
```

更多信息参考 [System time#Time standard](https://wiki.archlinux.org/title/System_time#Time_standard)。

## 本地化

`vim /etc/locale.gen` 写入以下内容: 

```bash
en_US.UTF-8 UTF-8
# zh_CN.UTF-8 UTF-8
```

生成 locale 信息: 

```bash
locale-gen
```

`vim /etc/locale.conf` 写入以下内容: 

```bash
LANG=en_US.UTF-8
# LANG=zh_CN.UTF-8
```

使用 `locale` 命令查看区域信息。

目前先使用 英文 locale, 等重启进入桌面之后再设置为中文 locale。虽然使用中文 locale 会导致 TTY 上中文显示为方块, 但是又不经常使用 TTY; 非要使用 TTY 的时候, 再手动设置英文 locale 就好了。

## btrfs内核模块

编辑 mkinitcpio 文件: 

```bash
vim /etc/mkinitcpio.conf
```

找到 `MODULES=()` 一行, 在括号中添加 `btrfs`。

这是为了在系统启动时提前加载 btrfs 内核模块, 从而正常启动系统。

{{< admonition type=info title="提示" open=true >}}
每次编辑 `/etc/mkinitcpio.conf` 后都需要运行 `mkinitcpio -P` 命令重新生成 initramfs。
{{< /admonition >}}

## GRUB

### 安装

```bash
pacman -S grub efibootmgr
```

参考 [GRUB](https://wiki.archlinux.org/title/GRUB) 安装: 

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

### 生成主配置文件

为防止得到如下报错: 

```
Warning: os-prober will not be executed to detect other bootable partitions
```

`vim /etc/default/grub`, 取消这一行的注释:

```
GRUB_DISABLE_OS_PROBER=false
```

如果没有这一行, 就在文件末尾加上。

运行 `os-prober` 命令探测其他系统。

生成主配置文件: 

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

{{< admonition type=info title="提示" open=true >}}
每次编辑 `/etc/default/grub` 后都需要运行 `grub-mkconfig -o /boot/grub/grub.cfg` 命令重新生成 GRUB 主配置文件。

若要管理多个 GRUB 条目, 比如既使用 `linux` 又使用 `linux-lts` 内核, 参考 [GRUB/Tips and tricks#Multiple entries](https://wiki.archlinux.org/title/GRUB/Tips_and_tricks#Multiple_entries)。
{{< /admonition >}}

## 用户设置

### Root用户

设置密码: 

```bash
passwd
```

### 普通用户

#### 创建

```bash
sudo useradd -d /home/<UserName> -m -s /bin/bash <UserName>
```

#### 设置密码

```bash
passwd <UserName>
```

#### 加入sudoers

`vim /etc/sudoers`, 在 `root ALL=(ALL:ALL) ALL` 下添加一行, 比如:

```bash
root ALL=(ALL:ALL) ALL
<UserName> ALL=(ALL:ALL) ALL
```

## 其他仓库

### 启用32位支持库

编辑 `/etc/pacman.conf`, 取消以下两行的注释: 

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

### 添加中文社区库

此处只能添加一个 Server, 参考 [archlinuxcn/mirrorlist-repo](https://github.com/archlinuxcn/mirrorlist-repo#readme), 选择一个镜像站, 虽然它列举的镜像站不全, 但大部分 arch 镜像站也同时托管 archlinuxcn, 可以自己检查一下。

编辑 `/etc/pacman.conf`, 添加: 

```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
# Server = https://mirrors.cqu.edu.cn/archlinuxcn/$arch
```

根据 Arch Linux CN 于 2023-12-18 发布的 [公告](https://www.archlinuxcn.org/archlinuxcn-keyring-manually-trust-farseerfc-key/), 新系统中安装 `archlinuxcn-keyring` 包前需要手动信任 **farseerfc** 的 key。

{{< admonition type=quote title="原因" open=true >}}
archlinuxcn 社区源的 keyring 包 archlinuxcn-keyring 由 farseerfc 的 key 签署验证, 而 Arch Linux 官方 keyring 中包含了 farseerfc 的 key。
[自12月初 archlinux-keyring 中删除了一个退任的 master key](https://gitlab.archlinux.org/archlinux/archlinux-keyring/-/issues/246) 导致 farseerfc 的 key 的信任数不足, 由 GnuPG 的 web of trust 推算为 marginal trust, 从而不再能自动信任 archlinuxcn-keyring 包的签名。
{{< /admonition >}}

如果你在新系统中尝试安装 `archlinuxcn-keyring` 包时遇到如下报错: 

```
error: archlinuxcn-keyring: Signature from "Jiachen YANG (Arch Linux Packager Signing Key) <farseerfc@archlinux.org>" is marginal trust
```

请使用以下命令在本地信任 **farseerfc** 的 key。此 key 已随 `archlinux-keyring` 安装在系统中, 只是缺乏信任: 

```bash
pacman-key --lsign-key "farseerfc@archlinux.org"
```

之后继续安装 `archlinuxcn-keyring`:

```
pacman -S archlinuxcn-keyring
```

## 重启

{{< admonition type=success title="快完成了🎉" open=true >}}
到这里, 绝大部分配置都已完成, 剩余的工作等重启以后再处理。
{{< /admonition >}}

1. 输入 `exit` 或按 `Ctrl+D` 退出 `chroot` 环境。
2. 可选用 `umount -R /mnt` 手动卸载被挂载的分区: 这有助于发现任何“繁忙”的分区, 并通过 [fuser](https://man.archlinux.org/man/fuser.1) 查找原因。
3. 输入 `reboot` 重启系统, systemd 将自动卸载仍然挂载的任何分区。
4. 不要忘记移除安装介质。
5. 使用 root 帐户登录到新系统。 

## 安装桌面

{{< admonition type=question title="确认Pacman相关配置" open=true >}}
忘记了上文中 [筛选镜像](#筛选镜像) 和 [Pacman配置](#Pacman配置) 这两个步骤的结果会不会保存到新系统了。可以先检查一下, 必要时再走一遍这两个步骤。
{{< /admonition >}}

然后更新系统: 

```bash
pacman -Syu
```

安装 [Xorg](https://wiki.archlinux.org/title/Xorg)、[SDDM](https://wiki.archlinux.org/title/SDDM)、[KDE Plasma](https://wiki.archlinux.org/title/KDE): 

```bash
pacman -S xorg sddm plasma
```

[xorg](https://archlinux.org/groups/x86_64/xorg/) 和 [plasma](https://archlinux.org/groups/x86_64/plasma/) 都是包组, 包组里的软件包大部分都有用, 全部安装即可。

安装过程会让用户选择依赖, 参考这则 [帖子](https://bbs.archlinuxcn.org/viewtopic.php?id=13151)。由于之后的 [音频](../2023-4/#音频) 设置使用 `PipeWire`, 所以这里选择带有 `PipeWire` 字样的依赖包。

设置 `sddm` 开机自启: 

```bash
systemctl enable sddm
```

## 后续工作

全部完成后重启。然后参考 [本系列](../series/use-arch/) 第 2 篇 [Arch配置](../2023-4/)。
