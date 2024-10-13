---
title: "Firefox代理开关"
date: 2024-01-10T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,Browser,CMD,Custom,PowerShell,Proxy]
featuredImagePreview: ""
summary: 通过修改 Firefox 配置文件 `user.js` 来修改其代理状态。
---

在 Firefox 中打开 `about:support` 页面, 在 `应用程序概要` 表中找到 `配置文件夹` 一栏, 点击 `打开目录` 来打开个人配置文件夹。

在 Firefox 图形界面中的设置数据会被存储到个人配置文件夹下的 `prefs.js` 中。比如在 Firefox 图形界面中的设置好代理后, `prefs.js` 中的代理设置: 

```javascript
user_pref("network.proxy.backup.ssl", "127.0.0.1");
user_pref("network.proxy.backup.ssl_port", 7890);
user_pref("network.proxy.http", "127.0.0.1");
user_pref("network.proxy.http_port", 7890);
user_pref("network.proxy.share_proxy_settings", true);
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", 7890);
user_pref("network.proxy.socks_remote_dns", true);
user_pref("network.proxy.ssl", "127.0.0.1");
user_pref("network.proxy.ssl_port", 7890);
user_pref("network.proxy.type", 1);
```

在个人配置文件夹中新建 `user.js` 文件, Firefox 每次启动时会使用 `user.js` 中的数据覆盖 `prefs.js`。

所以把覆写 `user.js` 代理设置的命令写入 Shell 的预加载文件中, 平时可以方便的输入命令来开关 Firefox 的代理了。不过执行完命令后需要重启 Firefox 才能生效。

{{< admonition type=warning title="不要更改 prefs.js" open=true >}}
如果在 Firefox 运行时直接更改了 `prefs.js` 中代理设置相关的值, 并不会立刻生效, 而且关闭 Firefox 时, Firefox 会将自己的设置写回 `prefs.js`, 所以改了也白改。

除非是在 Firefox 没有运行时更改 `prefs.js`, 那还不如直接改 `user.js`。
{{< /admonition >}}

## Bash

将自定义的 `function` 写入 [环境变量配置文件](../2021-6/) 中: 

```bash
function ff-proxy-on {
    cat <<EOF >"/home/<UserName>/.mozilla/firefox/xxx/user.js"
user_pref("network.proxy.backup.ssl", "127.0.0.1");
user_pref("network.proxy.backup.ssl_port", 7890);
user_pref("network.proxy.http", "127.0.0.1");
user_pref("network.proxy.http_port", 7890);
user_pref("network.proxy.share_proxy_settings", true);
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", 7890);
user_pref("network.proxy.socks_remote_dns", true);
user_pref("network.proxy.ssl", "127.0.0.1");
user_pref("network.proxy.ssl_port", 7890);
user_pref("network.proxy.type", 1);
EOF
    echo -e "Firefox Proxy \e[1;32menabled\e[0m. Restart Firefox to apply."
}

function ff-proxy-off {
    echo 'user_pref("network.proxy.type", 0);' >"/home/<UserName>/.mozilla/firefox/xxx/user.js"
    echo -e "Firefox Proxy \e[1;31mdisabled\e[0m. Restart Firefox to apply."
}
```

## CMD

参考 [这里](../2020-1/#永久代理), 将自定义的 `DOSKEY` 命令写入 `cmdrc.cmd` 文件中: 

```batch
@echo off
DOSKEY ff-proxy-on=echo user_pref^("network.proxy.type", 1^); ^> "%APPDATA%\Mozilla\Firefox\Profiles\xxx\user.js" $t echo Firefox Proxy enabled. Restart Firefox to apply.
DOSKEY ff-proxy-off=echo user_pref^("network.proxy.type", 0^); ^> "%APPDATA%\Mozilla\Firefox\Profiles\xxx\user.js" $t echo Firefox Proxy disabled. Restart Firefox to apply.
```

这里的 `ff-proxy-on` 只是设置了 `network.proxy.type`, 没有操作其他项, 是因为当命令很多时, 用 `DOSKEY` 就不合适了, 不过这样也够用, 反正不会闲着没事乱改其他项。

非要像上面 Bash 那样, 操作全部的 `network.proxy` 项, 也有其他方法, 最简单的是直接拿另外的配置文件覆盖 `user.js`: 

在 Firefox 个人配置文件夹下新建 `ff-proxy-on.js`, 写入本文开头的 JavaScript 代码, 然后, 将自定义的 `DOSKEY` 命令写入 `cmdrc.cmd` 文件中: 

```batch
@echo off
DOSKEY ff-proxy-on=xcopy /y "%APPDATA%\Mozilla\Firefox\Profiles\xxx\ff-proxy-on.js" "%APPDATA%\Mozilla\Firefox\Profiles\xxx\user.js" $t echo Firefox Proxy enabled. Restart Firefox to apply.
DOSKEY ff-proxy-off=echo user_pref^("network.proxy.type", 0^); ^> "%APPDATA%\Mozilla\Firefox\Profiles\xxx\user.js" $t echo Firefox Proxy disabled. Restart Firefox to apply.
```

## PowerShell

参考 [这里](../2020-1/#永久代理-1), 将自定义的 `function` 写入 `profile.ps1` 文件中: 

```powershell
function ff-proxy-on {
@'
user_pref("network.proxy.backup.ssl", "127.0.0.1");
user_pref("network.proxy.backup.ssl_port", 7890);
user_pref("network.proxy.http", "127.0.0.1");
user_pref("network.proxy.http_port", 7890);
user_pref("network.proxy.share_proxy_settings", true);
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", 7890);
user_pref("network.proxy.socks_remote_dns", true);
user_pref("network.proxy.ssl", "127.0.0.1");
user_pref("network.proxy.ssl_port", 7890);
user_pref("network.proxy.type", 1);
'@ | Out-File -FilePath "$env:APPDATA\Mozilla\Firefox\Profiles\xxx\user.js" -Encoding UTF8
    Write-Host "Firefox Proxy enabled. Restart Firefox to apply."
}

function ff-proxy-off {
    'user_pref("network.proxy.type", 0);' | Out-File -FilePath "$env:APPDATA\Mozilla\Firefox\Profiles\xxx\user.js" -Encoding UTF8
    Write-Host "Firefox Proxy disabled. Restart Firefox to apply."
}
```
