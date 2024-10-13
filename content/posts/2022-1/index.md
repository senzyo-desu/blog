---
title: "Windows自用设置"
date: 2022-01-03T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Windows]
featuredImagePreview: ""
summary: 备忘, 用于系统重装后进行自定义设置。
---

> 用于系统重装后的快速设置。

## WhyNotWin11

检测是否支持升级 Windows 11: [https://github.com/rcmaehl/WhyNotWin11/releases/latest](https://github.com/rcmaehl/WhyNotWin11/releases/latest)

## 下载ISO

### 从MSDN下载

更推荐从 MSDN 下载 ISO。因为 MSDN 的 ISO 是集成了最近的重要更新的, 所以版本比较新, 相比之下, 微软官网似乎只发布大版本的 ISO。

MSDN 官网: https://next.itellyou.cn/

### 从微软官网下载

最好直接下载 ISO, 而不是通过 `创建工具` 下载。

Windows 11: [https://www.microsoft.com/zh-cn/software-download/windows11](https://www.microsoft.com/zh-cn/software-download/windows11)

Windows 10: [https://www.microsoft.com/zh-cn/software-download/windows10](https://www.microsoft.com/zh-cn/software-download/windows10)

## 重装时注意事项

{{< admonition type=quote title="什么是OOBE" open=true >}}
客户首次打开 Windows 电脑时, 将看到 Windows 开箱体验 (OOBE)。 OOBE 是一系列屏幕, 这些屏幕需要客户接受许可协议、连接 Internet、登录或注册 Microsoft 帐户 (MSA) 并与 OEM 共享信息。 你在硬件和软件方面所做的选择确定了客户在享用新设备之前, 完成 OOBE 所需的工作量。
{{< /admonition >}}

### 跳过联网过程

如果想跳过联网激活过程, 或者想使用本地账户而不使用 Microsoft 账户, 需要在 OOBE 界面按下快捷键 `Shift+F10` 或者 `Shift+Fn+F10` 打开命令提示符, 输入 `oobe\bypassnro`, 电脑会自动重启, 重复 OOBE 界面, 这次就出现 `我没有 Internet 连接` 的按钮了。

### 禁用BitLocker

{{< admonition type=info title="仅影响 Windows 11" open=true >}}
根据 [蓝点网](https://www.landiannews.com/archives/103718.html) 消息: 

在 2023 年 7 月微软发布 Windows 11 Build 25905 版, 在此版本中微软已经调整启用设备加密的先决条件, 以便在全新安装 Windows 11 时自动启用设备加密功能。

最初该功能仅在 Windows 11 专业版中启用, 到今年 10 月份发布 Windows 11 24H2 正式版时, 预计家庭版系统也会默认采用设备加密功能。
{{< /admonition >}}

在 OOBE 界面按下快捷键 `Shift+F10` 或者 `Shift+Fn+F10` 打开命令提示符, 输入 `regedit` 打开注册表, 然后转到如下路径: 

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\BitLocker
```

在这里新建 `DWORD32` 位值并将其重命名为 `PreventDeviceEncryption`, 然后将这个新注册表项键值从 `0` 修改为 `1` 即可。

## 激活系统

[Microsoft Activation Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts/releases/latest) 适用于 Windows 8.1 至 Windows 11 以及 Microsoft Office。

## 禁止系统更新自动更新驱动

有时候自己安装的更新比 Windows 更新的时候安装的驱动更好、更新, 为了防止 Windows 更新将驱动换掉, 需要禁用自动驱动更新: 

1. 按下 `Win+S/Q`, 搜索并打开 `查看高级系统设置`。
2. 在 `高级系统设置` 的 `硬件` 选项卡下, 点击 `设备安装设置`。
3. 选择 `否, 每次只为我安装最兼容的驱动程序` 并保存更改。

然后通过本地组策略编辑器修改 Windows Update 的行为: 

1. 按下 `Win+S/Q`, 搜索并打开 `编辑组策略`。
2. 转到 `计算机配置`→`管理模板`→`Windows 组件`→`Windows 更新`。
3. 找到 `Windows 更新不包括驱动程序`, 启用它。

## 下载软件

- [7-zip](https://www.7-zip.org/)
- [AIDA64](https://www.aida64.com/downloads), 检查电脑参数
- [AM-DeadLink](https://www.aignes.com/deadlink.htm), 检查 URL 状态
- [Android Platform Tools](https://developer.android.com/studio/releases/platform-tools)
- [AppContainer Loopback Exemption Utility](https://www.telerik.com/fiddler/add-ons), 让 UWP 软件走代理
- [Aria2](https://github.com/aria2/aria2/releases/latest)
- [Autoruns](https://learn.microsoft.com/en-us/sysinternals/downloads/autoruns), 管理 Windows 自启动项
- [Bulk Crap Uninstaller](https://github.com/Klocman/Bulk-Crap-Uninstaller/releases/latest), 软件的安装监控与卸载清理
- [Context Menu Manager](https://gitee.com/BluePointLilac/ContextMenuManager), 管理 Windows 右键菜单
- [CrystalDiskInfo & CrystalDiskMark](https://crystalmark.info/en/download/), 硬盘的信息和测速
- [DoNotSpy11](https://pxc-coding.com/donotspy11/donotspy-11-download/), 启用和禁用 Windows 隐私项
- [dupeGuru](https://github.com/arsenetar/dupeguru/releases/latest), 查找重复文件
- [EasyUEFI](https://www.easyuefi.com/index-us.html), 管理 UEFI
- [Everything](https://www.voidtools.com/zh-cn/), 文件搜索
- [FFmpeg](https://www.ffmpeg.org/download.html)
- [FileZilla](https://filezilla-project.org/), FTP 服务器与客户端
- [Geek Uninstaller](https://geekuninstaller.com/), 软件的卸载清理
- [Git](https://git-scm.com/downloads)
- [Google Chrome](https://www.google.cn/intl/zh-CN/chrome/?standalone=1) 离线安装版
- [HEVC扩展​](https://store.rg-adguard.net/), 来自设备制造商的 HEVC 扩展。​ProductId: `9n4wgh0z6vhq`
- [HiBit Uninstaller](https://www.hibitsoft.ir/), 软件的安装监控与卸载清理
- [Hugo](https://github.com/gohugoio/hugo/releases/latest), 静态博客构建。下载 `hugo_extended`
- [InControl](https://www.grc.com/incontrol.htm), 控制 Windows 更新。可以只更新补丁, 不更新版本
- [Joplin](https://joplinapp.org/help/#desktop-applications), Markdown 笔记
- [Kaspersky](https://my.kaspersky.com/)
- [Kodi](https://kodi.tv/), 媒体娱乐中心
- [LocalSend](https://localsend.org/#/download), 局域网文件传输
- [LockHunter](https://lockhunter.com/), 检查占用文件的进程
- [Lunacy](https://icons8.com/lunacy), Figma 和 Sketch 的平替
- [Microsoft 365 E5 Renew Plus](https://e5renew.com/), 刷 Microsoft 365 E5 账户的活跃度
- [Microsoft .NET SDK](https://dotnet.microsoft.com/zh-cn/download/dotnet)
- [Microsoft Visual C++ Redistributable](https://learn.microsoft.com/zh-CN/cpp/windows/latest-supported-vc-redist)
- [Microsoft Visual Studio Code](https://code.visualstudio.com/#alt-downloads)
- [MusicPlayer2](https://github.com/zhongyang219/MusicPlayer2/releases/latest)
- [Node.js](https://nodejs.org/en/download/)
- [OBS Studio](https://obsproject.com/download)
- [Office Tool Plus](https://otp.landian.vip/zh-cn/) + [KMS](https://www.coolhub.top/tech-articles/kms_list.html), 安装 Microsoft Office
- [OpenJDK Archived](https://jdk.java.net/archive/)
- [Oracle JDK Archive](https://www.oracle.com/java/technologies/downloads/archive/)
- [Pandoc](https://github.com/jgm/pandoc/releases/latest), 转换文件格式
- [PixPin](https://pixpinapp.com/), 相比侧重于贴图的 Snipaste 有更多功能
- [pot-app](https://github.com/pot-app/pot-desktop/releases/latest), 语言翻译
- [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer), 管理软件进程
- [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon), 监控软件进程
- [QtScrcpy](https://github.com/barry-ran/QtScrcpy/releases/latest), 基于 ADB 的 Android 屏幕共享
- [QQ](https://im.qq.com/pcqq)
- [Resource Hacker](http://angusj.com/resourcehacker/), 修改 DLL 中的资源文件
- [Scrcpy](https://github.com/Genymobile/scrcpy/releases/latest), 基于 ADB 的 Android 屏幕共享
- [Snipaste](https://zh.snipaste.com/), 屏幕截图与贴图
- [Telegram](https://telegram.org/)
- [TinyTask](https://tinytask.net/), 只需录制动作无需编辑脚本的自动化工具
- [TopMost](https://www.sordum.org/9182/window-topmost-control-v1-2/), 置顶软件的窗口
- [TrafficMonitor](https://gitee.com/zhongyang219/TrafficMonitor), 在托盘显示 CPU、内存和网速等监控信息
- [TSAC](https://bellard.org/tsac/), 极低比特率但几乎不损失质量的音频压缩工具
- [ValiDrive](https://www.grc.com/validrive.htm), 检测 U 盘、存储卡、移动硬盘的真实容量, 避免虚标
- [Ventoy](https://github.com/ventoy/Ventoy/releases/latest), 为 ISO 等文件创建可启动的USB驱动器
- [Victoria HDD](https://hdd.by/victoria/), 检查机械硬盘的坏块
- [Vim](https://www.vim.org/download.php)
- [VLC](https://www.videolan.org/vlc/), 顺便一提 Android 版本在 [Play Store](https://play.google.com/store/apps/details?id=org.videolan.vlc) 停止更新, 但在其自托管的 Git [仓库](https://code.videolan.org/videolan/vlc-android/-/artifacts) 中持续更新
- [Windows 11 Classic Context Menu](https://www.sordum.org/14479/windows-11-classic-context-menu-v1-1/), 让 Windows 11 使用 Windows 10 样式的右键菜单
- [Windows Update Blocker](https://www.sordum.org/9470/windows-update-blocker-v1-8/), 启用和禁用 Windows 更新
- [Windows Defender Remover](https://github.com/ionuttbara/windows-defender-remover/releases/latest), 禁用或移除 Windows Defender 等安全组件
- [WizTree](https://diskanalyzer.com/download), 分析磁盘文件大小
- [阿里云盘](https://www.aliyundrive.com/)
- [百度网盘](https://pan.baidu.com/download)
- [哔哩下载姬](https://github.com/leiurayer/downkyi/releases/latest)
- [微信](https://weixin.qq.com/)

## 托盘时间显示秒数

{{< admonition type=success title="好消息" open=true >}}
较新版本的 Windows 11 在设置中直接添加了这个功能的开关, 用户不必手动编辑注册表了:

设置→个性化→任务栏→任务栏行为→在系统栏托盘时钟中显示秒数 (耗电更多)
{{< /admonition >}}

定位到注册表 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced`, 新建 `DWORD (32位) 值`, 命名为 `ShowSecondsInSystemClock`, 修改数值数据为 **1** (十六进制), 最后重启 **explorer.exe**。

此项功能 Windows 10 和 Windows 11 可用。值得注意的是, Windows 11 一开始删除了这项功能, 后来又把它加回来了。如果你设置后没有生效, 请将 Windows 11 更新至最新。

## 右键菜单检测文件哈希值

为什么要分成 Windows 10 和 Windows 11 两个部分呢, 因为我没能成功地自定义 Windows Terminal 打开后的窗口大小 (虽然翻了官方文档, 也看了 GitHub Issues)。

### Windows 10

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
"MUIVerb"="Get-FileHash"
"SubCommands"=""
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

; MD5
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5]
"MUIVerb"="MD5"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm MD5 | Format-List\""

;SHA1
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1]
"MUIVerb"="SHA1"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm SHA1 | Format-List\""

;SHA256
[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256]
"MUIVerb"="SHA256"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256\command]
@="PowerShell -NoExit -Command \"mode con cols=90 lines=15\";\"Get-FileHash '%1' -Algorithm SHA256 | Format-List\""
```

### Windows 11

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
"MUIVerb"="Get-FileHash"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"
"SubCommands"=""

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell]

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5]
"MUIVerb"="MD5"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\MD5\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm MD5 | Format-List"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1]
"MUIVerb"="SHA1"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA1\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm SHA1 | Format-List"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256]
"MUIVerb"="SHA256"
"Icon"="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe,0"

[HKEY_CLASSES_ROOT\*\shell\Get-FileHash\Shell\SHA256\command]
@="PowerShell -NoExit Get-FileHash '%1' -Algorithm SHA256 | Format-List"
```

### 移除

将以下内容写入一个 `txt` 中, 修改文件后缀名保存为 `.reg` 文件, 双击导入到注册表: 

```
Windows Registry Editor Version 5.00

[-HKEY_CLASSES_ROOT\*\shell\Get-FileHash]
```
