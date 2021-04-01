---
title: "设置CMD和PowerShell中的Alias"
date: 2021-03-21T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [CMD,PowerShell,Windows]
featuredImagePreview: ""
summary: 使用 `DOSKEY` 设置 CMD 中的 Alias, 使用 `New-Alias` 设置 PowerShell 中的 Alias, 将设置 Alias 的命令保存在预加载文件中来达到别名持久化。
---

## CMD

### DOSKEY

参考微软[官方文档](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/doskey), 使用`DOSKEY`命令设置别名: 

```batch
DOSKEY ls=dir /p $*
```

{{< admonition type=note title="注意" open=true >}}
1. `$*`的作用是替换参数, 比如`ls 1.txt`等效于`dir /p 1.txt`。
2. 多条命令用`$t`连接, 另外`$t`的前后不需要有空格, 可以直接连接两条命令: command1`$t`command2。
3. 在当前窗口设置的别名只在当前窗口有效。
{{< /admonition >}}

### 持久化

参考[这里](../2020-1/#永久代理), 将自定义的`DOSKEY`命令写入`cmdrc.cmd`文件中, 比如: 

```batch
@echo off

:: Aliases
DOSKEY cat=type $*
DOSKEY cd=cd /d $*
DOSKEY clear=cls
DOSKEY cp=xcopy /h /k $*
DOSKEY dns=ipconfig /flushdns
DOSKEY ls=dir /p $*
DOSKEY mv=move $*
DOSKEY proxy-set=set http_proxy=http://127.0.0.1:7890 $t set https_proxy=http://127.0.0.1:7890 $t echo Proxy environment variable has been set.
DOSKEY proxy-unset=set http_proxy= $t set https_proxy= $t echo Proxy environment variable has been unset.
```

## PowerShell

### Set-Alias

参考微软[官方文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/set-alias), 使用`Set-Alias`命令设置新的别名或修改已有别名: 

```shell
Set-Alias -Name 别名 -Value 原命令
```

比如: 

```shell
Set-Alias -Name aria2c -Value "aria2c -c -s16 -x16 -k1M"
```

> 也可以直接使用`Set-Alias`的别名`sal`。

### Get-Alias

使用`Get-Alias`命令查看所有别名, 也可以直接使用`Get-Alias`的别名`gal`。

### New-Alias

参考微软[官方文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_aliases#creating-an-alias), 使用`New-Alias`命令设置新的别名, 如果别名已存在会提示重复: 

```shell
New-Alias -Name 别名 -Value 原命令
```

> 也可以直接使用`Newt-Alias`的别名`nal`。

{{< admonition type=warning title="注意" open=true >}}
在当前窗口设置的别名只在当前窗口有效。
{{< /admonition >}}

### 持久化

不推荐使用[Export-Alias](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/export-alias)和[Import-Alias](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-alias), 太不方便了, 直接参考[这里](../2020-1/#永久代理-1), ~~将 Set-Alias -Name aria2c -Value "aria2c -c -s16 -x16 -k1M" 命令写入 PROFILE 文件中~~, 不能将这种长命令当作别名, 不然终端会报错。应该在`PROFILE`文件中自定义`function`:

```shell
function aria2c { & 'aria2c.exe' -c -s16 -x16 -k1M @args }
```
