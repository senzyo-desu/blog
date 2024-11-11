---
title: "sing-box on Windows"
date: 2024-01-09T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Proxy,Windows]
series: [Use sing-box]
series_weight: 3
featuredImagePreview: ""
summary: 开机自动更新 `config.json` 并启动 sing-box。通过任务计划程序和 Dashboard 网页来控制 sing-box。
---

{{< admonition type=info title="注意" open=true >}}
此文仅适用于客户端, 不适用于服务器和路由器。
{{< /admonition >}}

参考上篇 [sing-box on Linux](../2024-2/) 中的思路, 通过任务计划程序 `sing-box-trigger` 来开机自启动脚本 `Update.ps1`, 这个脚本将脚本循环检测网络可用性, 当网络可用时, 从远端下载配置文件替换本地的配置文件, 然后启动任务计划程序 `sing-box`。

## Core

从 https://github.com/SagerNet/sing-box/releases/ 下载, 解压到 sing-box 工作目录下, 比如 `C:\Users\<UserName>\Apps\sing-box\sing-box.exe`。

## Dashboard

创建 sing-box 工作目录: `C:\Users\<UserName>\Apps\sing-box`。

下载 [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/refs/heads/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip), 解压后将文件夹重命名为 `ui`, 移动到 sing-box 工作目录下。

## Task Scheduler

### sing-box-trigger

打开任务计划程序, 点击右侧操作列表中的“创建任务”:

1. 常规

   名称填 `sing-box-trigger`。
   
   安全选项中, 不用 `更改用户或组`, 使用默认的用户就行, 如果使用 `Administrators` 或者 `SYSTEM` 账户, 任务计划程序启动时会弹出运行窗口。

   选择 `不管用户是否登录都要运行`, 勾选 `不存储密码`。不然任务计划程序启动时也会弹出运行窗口。

   勾选 `使用最高权限运行`。

2. 触发器

    新建, `开始任务` 选择 `登录时` 或 `启动时`, 按需选择 `延迟任务时间`, 确认勾选 `已启用`。

3. 操作

    新建, `操作` 选择 `启动程序`; `程序或脚本` 填写 `powershell.exe`; `添加参数` 填写 `-ExecutionPolicy Bypass -File "Update.ps1"`; `起始于` 填写脚本所在目录, 比如 `C:\Users\<UserName>\Apps\sing-box`。

4. 条件

    全部取消勾选, 包括灰显的。

5. 设置

   1. 仅勾选 `允许按需运行任务`。
   2. 最下方的 `如果此任务已经运行, 以下规则适用`, 确认选择 `请勿启动新实例`。

### sing-box

打开任务计划程序, 点击右侧操作列表中的“创建任务”:

1. 常规

   名称填 `sing-box`。
   
   安全选项中, `更改用户或组`, 输入 `system`, 点击 `检查名称` 按钮, 变为 <u>SYSTEM</u>, 即使用 `SYSTEM` 账户。

2. 不使用触发器
3. 操作

    新建, `操作` 选择 `启动程序`; `程序或脚本` 填写 `sing-box.exe`; `添加参数` 填写 `-D .\ -c .\config.json run`; `起始于` 填写 sing-box 工作目录, 比如 `C:\Users\<UserName>\Apps\sing-box`。

4. 条件

    全部取消勾选, 包括灰显的。

5. 设置

   1. 仅勾选 `允许按需运行任务`。
   2. 最下方的 `如果此任务已经运行, 以下规则适用`, 确认选择 `请勿启动新实例`。

## 日常使用

关于脚本的设置, 参考 [关于脚本](../2023-7/#关于脚本) 和 [FAQ](../2023-7/#faq)。

### 更新配置文件

按需修改 `Update.ps1` 的内容:

更改 `$url_sub` 的值为自己的订阅链接, 至于模板 `$url_tpl`, 可以参考 [Generate config of sing-box](../2024-1/) 选择更多模板。

{{< admonition type=question title="无法启动?" open=false >}}
对于 sing-box 的运行参数, 由 `sing-box help` 得知: 

```
Flags:
  -c, --config stringArray             set configuration file path
  -C, --config-directory stringArray   set configuration directory path
  -D, --directory string               set working directory
      --disable-color                  disable color output
  -h, --help                           help for sing-box
```

`-C` 指定了配置文件所在文件夹的路径, 但没有指定使用哪一个配置文件; `-c` 则指明了使用的配置文件的路径。

如果在 [Service](#service) 中 `ExecStart` 部分使用的 sing-box 运行参数是  `-D .\ -C .\ run`。
则要求配置文件所在的文件夹中不能有多个 `json` 文件, 否则 sing-box 无法判定使用哪一个作为配置, 继而无法启动。
{{< /admonition >}}

{{< admonition type=tip title="提示" open=true >}}
- 从下面两种方案中选择一个即可。
- 手动运行这个更新配置文件的脚本时需要“以管理员身份运行”。
- 在系统环境变量 `PATH` 中添加 `C:\Users\<UserName>\Apps\sing-box\` 以便在脚本中方便地调用 `sing-box.exe`。
{{< /admonition >}}

#### 一种配置

下载使用 `mixed` 或 `tun` 入站的配置文件。

修改以下内容, 写入到脚本 `C:\Users\<UserName>\Apps\sing-box\Update.ps1`:

```shell
$task = "sing-box"
$ping_site = "ntp.aliyun.com"
$dir_config = "$env:USERPROFILE\Apps\sing-box"
$url_gene = "https://example.com"  # 生成配置的后端地址
$url_sub = "https://example.com"   # 来自机场的订阅链接
$inbound = "tun"                 # mixed or tun
$url_tpl = "https://raw.githubusercontent.com/senzyo-desu/sing-box-templates/normal/$inbound/dot/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
$url_dl = "$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"
$proxy_port = "7890"

Stop-ScheduledTask -TaskName $task
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
Clear-DnsClientCache
while (!(Test-Connection -ComputerName $ping_site -Count 1 -Quiet)) {
  Start-Sleep -Seconds 2
}
Set-Location -Path $dir_config
$count = 0
while ($count -lt 3) {
    Invoke-WebRequest -OutFile temp.json -Uri "$url_dl" -TimeoutSec 40
    sing-box.exe check -c temp.json 2>$null
    if ($LASTEXITCODE -eq 0) {
        Write-Host "Downloaded config($inbound) is valid."
        Remove-Item "config_$inbound.json" 2>$null
        Rename-Item temp.json "config_$inbound.json"
        Write-Host "Use config($inbound)."
        Copy-Item "config_$inbound.json" config.json
        break
    }
    Remove-Item temp.json 2>$null
    $count++
    Write-Host "Downloaded config($inbound) is invalid. Tried $count times."
}
if ($count -eq 3) {
    Write-Host "No changes, keep the existing files."
}
Start-ScheduledTask -TaskName $task
Start-Sleep -Seconds 5
$state = Get-ScheduledTask -TaskName $task | Select-Object -ExpandProperty State
Write-Host "State of ${task}: ${state}"
if ($state -eq "Running") {
    if ($inbound -eq "mixed") {
        Write-Host "Use system proxy mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxy_port"
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
    } elseif ($inbound -eq "tun") {
        Write-Host "Use tun mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
    }
} else {
    Write-Host "Run ${task} failed."
}
Start-Sleep -Seconds 1
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行任务计划程序 `sing-box-trigger` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

#### 两种配置

下载使用 `mixed` 和 `tun` 入站的配置文件。优先使用 `tun` 入站。搭配 [切换模式重启](#切换模式重启) 脚本来切换模式。

修改以下内容, 写入到脚本 `C:\Users\<UserName>\Apps\sing-box\Update.ps1`:

```shell
$task = "sing-box"
$ping_site = "ntp.aliyun.com"
$dir_config = "$env:USERPROFILE\Apps\sing-box"
$url_gene = "https://example.com"  # 生成配置的后端地址
$url_sub = "https://example.com"   # 来自机场的订阅链接
$inbound_prefer = "tun"          # mixed or tun
$proxy_port = "7890"

Stop-ScheduledTask -TaskName $task
Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
Clear-DnsClientCache
while (!(Test-Connection -ComputerName $ping_site -Count 1 -Quiet)) {
  Start-Sleep -Seconds 2
}
Set-Location -Path $dir_config
$inbound_list = @("mixed", "tun")
foreach ($inbound in $inbound_list) {
    $url_tpl = "https://raw.githubusercontent.com/senzyo-desu/sing-box-templates/normal/$inbound/dot/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
    $url_dl = "$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"
    $count = 0
    while ($count -lt 3) {
        Invoke-WebRequest -OutFile temp.json -Uri "$url_dl" -TimeoutSec 40
        sing-box.exe check -c temp.json 2>$null
        if ($LASTEXITCODE -eq 0) {
            Write-Host "Downloaded config($inbound) is valid."
            Remove-Item "config_$inbound.json" 2>$null
            Rename-Item temp.json "config_$inbound.json"
            if ($inbound -eq $inbound_prefer) {
                Write-Host "Use config($inbound)."
                Copy-Item "config_$inbound.json" config.json
            }
            break
        }
        Remove-Item temp.json 2>$null
        $count++
        Write-Host "Downloaded config($inbound) is invalid. Tried $count times."
    }
    if ($count -eq 3) {
        Write-Host "No changes, keep the existing files."
    }
}
Start-ScheduledTask -TaskName $task
Start-Sleep -Seconds 5
$state = Get-ScheduledTask -TaskName $task | Select-Object -ExpandProperty State
Write-Host "State of ${task}: ${state}"
if ($state -eq "Running") {
    if ($inbound_prefer -eq "mixed") {
        Write-Host "Use system proxy mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxy_port"
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
    } elseif ($inbound_prefer -eq "tun") {
        Write-Host "Use tun mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
    }
} else {
    Write-Host "Run ${task} failed."
}
Start-Sleep -Seconds 1
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行任务计划程序 `sing-box-trigger` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

### 当前配置重启

修改以下内容, 写入到脚本 `C:\Users\<UserName>\Apps\sing-box\Restart.ps1`:

```shell
$dir_config = "$env:USERPROFILE\Apps\sing-box"
$task = "sing-box"
$proxy_port = "7890"

Set-Location -Path $dir_config
sing-box.exe check -c config.json 2>$null
if ($LASTEXITCODE -ne 0) {
    Write-Host "File config.json is invalid."
    Start-Sleep -Seconds 2
    Exit 1
}
Stop-ScheduledTask -TaskName $task
Clear-DnsClientCache
Start-ScheduledTask -TaskName $task
Start-Sleep -Seconds 5
$state = Get-ScheduledTask -TaskName $task | Select-Object -ExpandProperty State
Write-Host "State of ${task}: ${state}"
if ($state -eq "Running") {
    $find = Select-String -Path config.json -Pattern '"type": "tun"'
    if ($find) {
        Write-Host "Use tun mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
        Write-Host "System proxy disabled."
    } else {
        Write-Host "Use system proxy mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxy_port"
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
        Write-Host "System proxy enabled."
    }
} else {
    Write-Host "Run ${task} failed."
}
Start-Sleep -Seconds 1
```

### 停止运行

修改以下内容, 写入到脚本 `C:\Users\<UserName>\Apps\sing-box\Stop.ps1`:

```shell
$task = "sing-box"

Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
Write-Host "System proxy disabled."
Stop-ScheduledTask -TaskName $task
Clear-DnsClientCache
$state = Get-ScheduledTask -TaskName $task | Select-Object -ExpandProperty State
Write-Host "State of ${task} ScheduledTask: ${state}"
Start-Sleep -Seconds 1
```

### 切换模式重启

sing-box 的入站方式是固定在配置文件中的, 而且不支持通过 API 切换, 所以只能切换配置文件来切换 `mixed` 或 `tun` 模式。

修改以下内容, 写入到脚本 `C:\Users\<UserName>\Apps\sing-box\Switch.ps1`:

```shell
$dir_config = "$env:USERPROFILE\Apps\sing-box"
$task = "sing-box"
$proxy_port = "7890"

Set-Location -Path $dir_config
$find = Select-String -Path config.json -Pattern '"type": "tun"'
if ($find) {
    $inbound = "mixed"
} else {
    $inbound = "tun"
}
sing-box.exe check -c config_$inbound.json 2>$null
if ($LASTEXITCODE -ne 0) {
    Write-Host "File config_$inbound.json is invalid."
    Start-Sleep -Seconds 2
    Exit 1
}
Stop-ScheduledTask -TaskName $task
Clear-DnsClientCache
Copy-Item "config_$inbound.json" config.json
Start-ScheduledTask -TaskName $task
Start-Sleep -Seconds 5
$state = Get-ScheduledTask -TaskName $task | Select-Object -ExpandProperty State
Write-Host "State of ${task}: ${state}"
if ($state -eq "Running") {
    if ($inbound -eq "mixed") {
        Write-Host "Use system proxy mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyServer -Value "127.0.0.1:$proxy_port"
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 1
        Write-Host "System proxy enabled."
    } else {
        Write-Host "Use tun mode."
        Set-ItemProperty -Path 'Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings' -Name ProxyEnable -Value 0
        Write-Host "System proxy disabled."
    }
} else {
    Write-Host "Run ${task} failed."
}
Start-Sleep -Seconds 1
```

## 附件

几个用于脚本快捷方式的自制图标:

<table>
  <tr>
    <td align="center"><img src="./Restart.svg" width="100px" /></td>
    <td align="center"><img src="./Stop.svg" width="100px" /></td>
    <td align="center"><img src="./Switch.svg" width="100px" /></td>
    <td align="center"><img src="./Update.svg" width="100px" /></td>
  </tr>
</table>
