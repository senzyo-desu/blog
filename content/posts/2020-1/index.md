---
title: "终端代理设置"
date: 2020-05-21T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,CMD,Git,PowerShell,Proxy]
featuredImagePreview: "/2020-1/featuredImagePreview.svg"
summary: Git, Linux 与 Windows 的终端代理设置。
---

## CMD

### 临时代理

{{< admonition type=note title="注意" open=true >}}
只对当前终端有效, 新建终端需要重新设置。
{{< /admonition >}}

设置代理 **格式**: 

```batch
set http_proxy=protocol://[proxy_userid]:[proxy_password]@proxy_ip:proxy_port
```

**HTTP代理**: 

```batch
set http_proxy=http://127.0.0.1:7890
set https_proxy=http://127.0.0.1:7890
```

**SOCKS5代理**: 

```batch
set http_proxy=socks5://127.0.0.1:7890
set https_proxy=socks5://127.0.0.1:7890
```

检查是否设置成功: 

```batch
echo %http_proxy%
echo %https_proxy%
```

### 取消代理

取消当前终端的代理设置: 

```batch
set http_proxy=
set https_proxy=
```

### 永久代理

#### 方法一: 默认开启代理

1. 新建一个`cmdrc.cmd`, 将上面相应的代理命令写入。
2. 打开注册表, 在`HKEY_LOCAL_MACHINE\Software\Microsoft\Command Processor\`下新建名为`AutoRun`的**字符串值**, 数据的值是一个绝对路径, 路径指向刚才新建的`cmdrc.cmd`, 比如`C:\Users\<UserName>\cmdrc.cmd`。
3. 此脚本会在每次打开cmd时被预加载。
4. 修改文件权限, 禁用继承 (转化为显式权限), 除`SYSTEM`和`Administrators`设为**完全控制**外, 其余用户均设为**读取和执行**。

#### 方法二: DOSKEY作为开关(推荐)

上面的[方法一: 默认开启代理](#方法一-默认开启代理), 终端一打开就默认启用了代理, 如要关闭还需输入一长串命令。有一种更好的办法: 在`cmdrc.cmd`中不默认启用代理, 而是设置灵活的别名作为代理的开关: 

```batch
@echo off
DOSKEY proxy-set=set http_proxy=http://127.0.0.1:7890 $t set https_proxy=http://127.0.0.1:7890 $t echo Proxy environment variable has been set.
DOSKEY proxy-unset=set http_proxy= $t set https_proxy= $t echo Proxy environment variable has been unset.
```

{{< admonition type=tip title="技巧" open=true >}}
`$t`的前后也可以不需要空格, 直接连接两条命令: command1`$t`command2。
{{< /admonition >}}

日常使用时, 输入`proxy-set`即可设置代理, 输入`proxy-unset`即可取消设置代理。

## PowerShell

### 临时代理

{{< admonition type=note title="注意" open=true >}}
只对当前终端有效, 新建终端需要重新设置。
{{< /admonition >}}

设置代理 **格式**: 

```shell
$env:http_proxy="protocol://[proxy_userid]:[proxy_password]@proxy_ip:proxy_port"
```

**HTTP代理**: 

```shell
$env:http_proxy="http://127.0.0.1:7890"
$env:https_proxy="http://127.0.0.1:7890"
```

{{< admonition type=question title="似乎不支持 SOCKS 代理" open=true >}}
PowerShell 似乎不支持以环境变量的方式设置 SOCKS 代理。

但可以确定的是, `Invoke-WebRequest` 在 PowerShell 5.1 只支持 HTTP 代理, 在 PowerShell 7.x 支持 HTTPS 和 SOCKS 代理。

使用示例:

```shell
Invoke-RestMethod -Proxy "socks5://127.0.0.1:7890" -Uri "https://td.telegram.org/tsetup/tsetup.5.1.5.exe" -OutFile "tsetup.5.1.5.exe"
```
{{< /admonition >}}

检查是否设置成功: 

```shell
$env:http_proxy
$env:https_proxy
```

### 取消代理

取消当前终端的代理设置: 

```shell
$env:http_proxy=""
$env:https_proxy=""
```

### 永久代理

#### 方法一: 默认开启代理

参考微软[官方文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles#how-to-create-a-profile), 为所有用户、所有主机生成`profile`文件: 

1. 在PowerShell中运行以下命令: 

    ```shell
    if (!(Test-Path -Path $PROFILE.AllUsersAllHosts)) { New-Item -ItemType File -Path $PROFILE.AllUsersAllHosts -Force };
    ```

    {{< admonition type=tip title="其他级别" open=true >}}
- Current User, Current Host - $PROFILE
- Current User, Current Host - $PROFILE.CurrentUserCurrentHost
- Current User, All Hosts - $PROFILE.CurrentUserAllHosts
- All Users, Current Host - $PROFILE.AllUsersCurrentHost
- All Users, All Hosts - $PROFILE.AllUsersAllHosts
    {{< /admonition >}}

2. 使用`notepad $PROFILE.AllUsersAllHosts`命令调用记事本打开`profile`文件, 在文件中写入上面相应的代理命令, 保存关闭即可, 该文件会在每次打开PowerShell时被预加载。
3. 如果是`Current User, Current Host`级别, 则`profile`文件为`此电脑/文档/WindowsPowerShell/Microsoft.PowerShell_profile.ps1`
4. 如果是`All Users, All Hosts`级别, 则`profile`文件为`C:\Windows\System32\WindowsPowerShell\v1.0\profile.ps1`
5. 如果在PowerShell 7生成`profile`文件, 则一律为`C:\Program Files\PowerShell\7\profile.ps1`
6. 修改文件权限, 禁用继承 (转化为显式权限), 除`SYSTEM`和`Administrators`设为**完全控制**外, 其余用户均设为**读取和执行**。

##### Windows不允许运行脚本? 

参考微软官方文档[Set-ExecutionPolicy](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.security/set-executionpolicy), 以管理员身份运行PowerShell, 执行以下命令: 

```shell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
```

#### 方法二: function作为开关(推荐)

上面的[方法一: 默认开启代理](#方法一-默认开启代理-1), 终端一打开就默认启用了代理, 如要关闭还需输入一长串命令。有一种更好的办法: 在`profile`文件中不默认启用代理, 而是设置灵活的`function`作为代理的开关: 

```shell
function proxy-set {
    $env:http_proxy="http://127.0.0.1:7890"
    $env:https_proxy="http://127.0.0.1:7890"
    Write-Host "Proxy environment variable has been set."
}

function proxy-unset {
    $env:http_proxy=""
    $env:https_proxy=""
    Write-Host "Proxy environment variable has been unset."
}
```

日常使用时, 输入`proxy-set`即可设置代理, 输入`proxy-unset`即可取消设置代理。

## Bash、Zsh等

### 临时代理

{{< admonition type=note title="注意" open=true >}}
只对当前终端有效, 新建终端需要重新设置。
{{< /admonition >}}

设置代理 **格式**: 

```bash
export http_proxy=protocol://[proxy_userid]:[proxy_password]@proxy_ip:proxy_port
```

**HTTP代理**: 

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

**SOCKS5代理**:

```bash
export http_proxy=socks5://127.0.0.1:7890
export https_proxy=socks5://127.0.0.1:7890
```

**全部代理**: 

```bash
export all_proxy=socks5://127.0.0.1:7890
```

检查是否设置成功: 

```bash
echo $http_proxy
echo $https_proxy
echo $all_proxy
```

### 取消代理

取消当前终端的代理设置: 

```bash
unset http_proxy https_proxy
unset all_proxy
```

### 永久代理

#### 方法一: 默认开启代理

将上面的临时代理命令写入 [环境变量配置文件](../2021-6/) 中。这样新建的终端默认会使用代理。

#### 方法二: alias作为开关(推荐)

通过设置 `alias` 来简化操作。将以下命令写入 [环境变量配置文件](../2021-6/) 中。

```bash
alias proxy-set="export http_proxy=http://127.0.0.1:7890 && export https_proxy=http://127.0.0.1:7890 && echo -e 'Proxy environment variable has been \e[1;32mset\e[0m.'"
alias proxy-unset="unset http_proxy https_proxy && echo -e 'Proxy environment variable has been \e[1;31munset\e[0m.'"
```

每次要使用代理就输入 `proxy-set`, 不用了就输入 `proxy-unset`。

#### 方法三: function作为开关(推荐)

通过设置 `function` 来简化操作。将以下命令写入 [环境变量配置文件](../2021-6/) 中。

```bash
function proxy-set {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=http://127.0.0.1:7890
    echo -e "Proxy environment variable has been \e[1;32mset\e[0m."
}

function proxy-unset {
    unset http_proxy https_proxy
    echo -e "Proxy environment variable has been \e[1;31munset\e[0m."
}
```

每次要使用代理就输入 `proxy-set`, 不用了就输入 `proxy-unset`。

## Git设置代理

### 永久代理

#### 方法一: 默认开启代理

{{< admonition type=note title="注意" open=true >}}
使用`--global`参数即全局永久生效, 不使用则对当前仓库永久生效, 相当于`--local`, 关于配置级别, 参考[Git Config](../2020-7/#配置级别)。
{{< /admonition >}}

**格式**: 

```shell
git config --global http.proxy protocol://[proxy_userid]:[proxy_password]@proxy_ip:proxy_port
```

**HTTP代理**: 

```shell
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

**SOCKS5代理**: 

```shell
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
```

##### 取消代理

```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

使用`git config --global --list`命令检查是否设置成功。

#### 方法二: 在各种Shell中使用Git

使用上文的各种永久代理方案来配置`cmd`, `PowerShell`, `bash`, `zsh`等Shell, 然后在这些Shell中使用Git, 这样Git也会走代理。

#### 方法三: Git Bash on Windows

修改`C:\Program Files\Git\etc\profile.d\aliases.sh`, 添加alias:

```bash
alias proxy-set="export http_proxy=http://127.0.0.1:7890 && export https_proxy=http://127.0.0.1:7890 && echo Proxy environment variable has been set."
alias proxy-unset="unset http_proxy https_proxy && echo Proxy environment variable has been unset."
```

该`aliases.sh`文件会在每次打开Git Bash时被自动加载, 所以每次要使用代理只需输入`proxy-set`, 不用了就输入`proxy-unset`即可。