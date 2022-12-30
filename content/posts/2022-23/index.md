---
title: "自定义PowerShell提示符并着色"
date: 2022-12-15T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,PowerShell,Windows]
series: [Customize Prompt of Shell]
series_weight: 2
featuredImagePreview: ""
summary: Customize and Colorize PowerShell Prompt.
---

## ANSI转义字符

和 Bash 类似, 要控制显示格式, 须使用 SGR 转义序列: `ESC[<n>m`。简单理解, 以 `ESC[` 开头, 中间 `<n>` 处设置颜色, 以 `m` 结尾。

颜色代码参考微软官方文档 [Text Formatting](https://learn.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#text-formatting)。

## PowerShell 7.x

支持以 `[char]27` 和 \``e` 表示 `ESC`。

对于 Prompt 的多行显示支持正常。

### 代码

参考 [这里](../2020-1/#永久代理-1) 设置永久配置 `$PROFILE`, 将下面的代码写入, 使其在打开 PowerShell 时可以被预加载。

```powershell
function prompt {
    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = [Security.Principal.WindowsPrincipal] $identity
    $adminRole = [Security.Principal.WindowsBuiltInRole]::Administrator

    "`r`n" + "`e[97m$(Get-Date -Format "HH:mm:ss")" + " " + "`e[91m$(if($principal.IsInRole($adminRole)) { "admin" } else { "senzyo" })" + "`e[97m@" + "`e[91m$env:computername" + " " + "`e[96m$(Get-Location)" + "`r`n" + "`e[91m$(if($principal.IsInRole($adminRole)) { "#" } else { "$" }) `e[0m"
}
```

### 解释

- [Get-Date](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-date) 输出当前日期和时间, `Get-Date -Format "HH:mm:ss"` 指定输出格式
- `if($principal.IsInRole($adminRole)` 以及 `$identity`, `$principal`, `$adminRole` 用于判断当前用户是否为管理员
- [Get-Location](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-location) 输出当前位置

### 效果

普通用户:

<div style="background-color:black">
    <br /><span style="color:white;">20:34:50</span> <span style="color:red;">senzyo</span><span style="color:white;">@</span><span style="color:red;">Windows</span> <span style="color:cyan;">D:\Desktop</span><div></div><span style="color:red;">$ </span>
</div>

管理员账户:

<div style="background-color:black">
    <br /><span style="color:white;">20:34:50</span> <span style="color:red;">admin</span><span style="color:white;">@</span><span style="color:red;">Windows</span> <span style="color:cyan;">D:\Desktop</span><div></div><span style="color:red;"># </span>
</div>

## PowerShell 5.x

支持以 `[char]27` 表示 `ESC`, 不以支持 \``e` 表示。

{{< admonition type=bug title="BUG" open=true >}}
PowerShell 5.x 对于 Prompt 的多行显示支持存在 BUG, 更高版本的 PowerShell 已修复此问题。

所以下面的样式只是单行。
{{< /admonition >}}

### 代码

参考 [这里](../2020-1/#永久代理-1) 设置永久配置 `$PROFILE`, 将下面的代码写入, 使其在打开 PowerShell 时可以被预加载。

```powershell
function prompt {
    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = [Security.Principal.WindowsPrincipal] $identity
    $adminRole = [Security.Principal.WindowsBuiltInRole]::Administrator
    $ESC = [char]27

    "$ESC[97m$(Get-Date -Format "HH:mm:ss")" + " " + "$ESC[91m$(if($principal.IsInRole($adminRole)) { "admin" } else { "senzyo" })" + "$ESC[97m@" + "$ESC[91m$env:computername" + " " + "$ESC[96m$(Get-Location)" + "$ESC[91m$(if($principal.IsInRole($adminRole)) { " #" } else { " $" }) $ESC[0m"
}
```

### 效果

普通用户:

<div style="background-color:black">
    <span style="color:white;">20:34:50</span> <span style="color:red;">senzyo</span><span style="color:white;">@</span><span style="color:red;">Windows</span> <span style="color:cyan;">D:\Desktop</span> <span style="color:red;">$ </span>
</div>

管理员账户:

<div style="background-color:black">
    <span style="color:white;">20:34:50</span> <span style="color:red;">admin</span><span style="color:white;">@</span><span style="color:red;">Windows</span> <span style="color:cyan;">D:\Desktop</span> <span style="color:red;"># </span>
</div>

--------------------

{{< admonition type=tip title="顺便一提" open=true >}}
PowerShell 后跟启动参数 `-nologo`, CMD 后跟启动参数 `/k`, 可以取消显示顶端的版本和版权信息。
{{< /admonition >}}

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [PowerShell Prompts](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_prompts)
2. [Text Formatting](https://learn.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#text-formatting)
{{< /admonition >}}
