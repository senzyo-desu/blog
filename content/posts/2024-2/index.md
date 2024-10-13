---
title: "sing-box on Linux"
date: 2024-01-08T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Proxy,Linux]
series: [Use sing-box]
series_weight: 2
featuredImagePreview: ""
summary: 开机自动更新 `config.json` 并启动 sing-box。通过 systemd 服务和 Dashboard 网页来控制 sing-box。
---

{{< admonition type=info title="注意" open=true >}}
此文仅适用于客户端, 不适用于服务器和路由器。
{{< /admonition >}}

思路是通过 [Systemd](https://wiki.archlinux.org/title/Systemd) 的 [Timer](https://wiki.archlinux.org/title/systemd/Timers) 来开机自启动对应的 Service, 这个 Service 将执行一个脚本, 脚本循环检测网络可用性, 当网络可用时, 从远端下载配置文件替换本地的配置文件, 然后启动 `sing-box.service`。

## Core

使用 sing-box [官方脚本](https://sing-box.sagernet.org/zh/installation/package-manager/) 安装的是 [正式版](https://github.com/SagerNet/sing-box/releases/latest), 要想使用 [预发行版](https://github.com/SagerNet/sing-box/releases/), 需要手动下载安装:

```
tar -zxvf sing-box-<version>-linux-amd64.tar.gz
mv sing-box /usr/local/bin/sing-box
```

## Dashboard

下载 [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/refs/heads/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip), 解压后将文件夹重命名为 `ui`。

```bash
mkdir /etc/sing-box
sudo cp -r ui /etc/sing-box/
```

## Service

创建 sing-box 的 systemd 服务: 

```bash
sudo vim /etc/systemd/system/sing-box.service
```

```
[Unit]
Description=sing-box service
Documentation=https://sing-box.sagernet.org
After=network.target nss-lookup.target

[Service]
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE CAP_SYS_PTRACE CAP_DAC_READ_SEARCH
ExecStart=/usr/local/bin/sing-box -D /etc/sing-box -c /etc/sing-box/config.json run
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10s
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
```

## Timer

```bash
sudo vim /etc/systemd/system/sing-box-trigger.timer
```

```
[Unit]
Description=sing-box trigger timer

[Timer]
OnBootSec=10s
Unit=sing-box-trigger.service

[Install]
WantedBy=timers.target
```

```bash
sudo vim /etc/systemd/system/sing-box-trigger.service
```

```
[Unit]
Description=sing-box trigger service

[Service]
Type=simple
ExecStart=/etc/sing-box/update.sh
Restart=on-watchdog
WatchdogSec=60
```

```bash
sudo systemctl enable sing-box-trigger.timer
```

```bash
sudo systemctl status sing-box-trigger.timer
```

## 日常使用

### 更新配置文件

按需修改 `update.sh` 的内容:

更改 `$url_sub` 的值为自己的订阅链接, 至于模板 `$url_tpl`, 可以参考 [Generate config of sing-box](../2024-1/) 选择更多模板。

脚本通过 KDE 的 `kwriteconfig6` 来控制系统代理的开关。更改 `$kioslaverc` 的值为自己的 `kioslaverc` 文件路径, 如不需要可将其删除并注释 `kwriteconfig6` 部分。

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

如果在 [Service](#service) 中 `ExecStart` 部分使用的 sing-box 运行参数是 `-D /etc/sing-box -C /etc/sing-box run`。
则要求配置文件所在的文件夹中不能有多个 `json` 文件, 否则 sing-box 无法判定使用哪一个作为配置, 继而无法启动。
{{< /admonition >}}

{{< admonition type=tip title="提示" open=true >}}
- 从下面两种方案中选择一个即可。
- 手动运行这个更新配置文件的脚本时需要 `sudo`。
{{< /admonition >}}

#### 一种配置

下载使用 `mixed` 或 `tun` 入站的配置文件。

```bash
vim /etc/sing-box/update.sh
```

```bash
#!/bin/bash

reset="\e[0m"
black="\e[1;30m"
red="\e[1;31m"
green="\e[1;32m"
yellow="\e[1;33m"
blue="\e[1;34m"
purple="\e[1;35m"
cyan="\e[1;36m"
white="\e[1;37m"

service="sing-box.service"
ping_site="ntp.aliyun.com"
dir_config="/etc/sing-box"
url_gene="https://example.com"  # 生成配置的后端地址
url_sub="https://example.com"   # 来自机场的订阅链接
inbound="tun"                   # mixed or tun
url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-templates/normal/$inbound/dot/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
url_dl="$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"
kioslaverc="/home/<UserName>/.config/kioslaverc"
proxy_port="7890"

sudo systemctl stop $service
kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
until ping -c 1 $ping_site &>/dev/null; do
    sleep 2
done
cd $dir_config
count=0
while [ $count -lt 3 ]; do
    curl -fsSLo temp.json "$url_dl" --connect-timeout 10 --max-time 40
    sing-box check -c temp.json &>/dev/null
    if [ $? -eq 0 ]; then
        echo -e "Downloaded ${yellow}config($inbound)${reset} is ${green}valid${reset}."
        mv temp.json config_$inbound.json
        echo -e "Use ${yellow}config($inbound)${reset}."
        cp config_$inbound.json config.json
        break
    fi
    rm -f temp.json
    ((count++))
    echo -e "Downloaded ${yellow}config($inbound)${reset} is ${red}invalid${reset}. Tried $count times."
done
if [ $count -eq 3 ]; then
    echo "No changes, keep the existing files."
fi
sudo systemctl start $service
sleep 5
sub_state=$(systemctl show -p SubState --value $service)
if [ $sub_state == "running" ]; then
    echo -e "$service state: ${green}$sub_state${reset}."
    if [[ $inbound == "mixed" ]]; then
        echo -e "Use ${blue}system proxy${reset} mode."
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
        echo -e "KDE proxy ${green}enabled${reset}."
    elif [[ $inbound == "tun" ]]; then
        echo -e "Use ${blue}tun${reset} mode."
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
        echo -e "KDE proxy ${red}disabled${reset}."
    fi
else
    echo -e "$service state: ${red}$sub_state${reset}."
fi
```

```bash
chmod +x /etc/sing-box/update.sh
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行 `sudo systemctl start sing-box-trigger.timer` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

#### 两种配置

下载使用 `mixed` 和 `tun` 入站的配置文件。优先使用 `tun` 入站。搭配 [切换模式重启](#切换模式重启) 函数来切换模式。

```bash
vim /etc/sing-box/update.sh
```

```bash
#!/bin/bash

reset="\e[0m"
black="\e[1;30m"
red="\e[1;31m"
green="\e[1;32m"
yellow="\e[1;33m"
blue="\e[1;34m"
purple="\e[1;35m"
cyan="\e[1;36m"
white="\e[1;37m"

service="sing-box.service"
ping_site="ntp.aliyun.com"
dir_config="/etc/sing-box"
url_gene="https://example.com"  # 生成配置的后端地址
url_sub="https://example.com"   # 来自机场的订阅链接
inbound_prefer="tun"          # mixed or tun
kioslaverc="/home/<UserName>/.config/kioslaverc"
proxy_port="7890"

sudo systemctl stop $service
kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
until ping -c 1 $ping_site &>/dev/null; do
    sleep 2
done
cd $dir_config
inbound_list=(mixed tun)
for inbound in "${inbound_list[@]}"; do
    url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-templates/normal/$inbound/dot/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
    url_dl="$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"
    count=0
    while [ $count -lt 3 ]; do
        curl -fsSLo temp.json "$url_dl" --connect-timeout 10 --max-time 40
        sing-box check -c temp.json &>/dev/null
        if [ $? -eq 0 ]; then
            echo -e "Downloaded ${yellow}config($inbound)${reset} is ${green}valid${reset}."
            mv temp.json config_$inbound.json
            if [[ $inbound == $inbound_prefer ]]; then
                echo -e "Use ${yellow}config($inbound)${reset}."
                cp config_$inbound.json config.json
            fi
            break
        fi
        rm -f temp.json
        ((count++))
        echo -e "Downloaded ${yellow}config($inbound)${reset} is ${red}invalid${reset}. Tried $count times."
    done
    if [ $count -eq 3 ]; then
        echo "No changes, keep the existing files."
    fi
done
sudo systemctl start $service
sleep 5
sub_state=$(systemctl show -p SubState --value $service)
if [ $sub_state == "running" ]; then
    echo -e "$service state: ${green}$sub_state${reset}."
    if [[ $inbound_prefer == "mixed" ]]; then
        echo -e "Use ${blue}system proxy${reset} mode."
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
        echo -e "KDE proxy ${green}enabled${reset}."
    elif [[ $inbound_prefer == "tun" ]]; then
        echo -e "Use ${blue}tun${reset} mode."
        kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
        echo -e "KDE proxy ${red}disabled${reset}."
    fi
else
    echo -e "$service state: ${red}$sub_state${reset}."
fi
```

```bash
chmod +x /etc/sing-box/update.sh
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行 `sudo systemctl start sing-box-trigger.timer` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

### 开关系统和浏览器代理

Chrome 会跟随 KDE 系统代理, Firefox 则不然。所以对于 KDE 系统代理和 [Firefox](../2024-4) 代理, 将操作命令封装成函数写入 [环境变量配置文件](../2021-6/) 中, 比如修改以下内容写入 `/etc/proxy-custom`: 

```bash
reset="\e[0m"
black="\e[1;30m"
red="\e[1;31m"
green="\e[1;32m"
yellow="\e[1;33m"
blue="\e[1;34m"
purple="\e[1;35m"
cyan="\e[1;36m"
white="\e[1;37m"

proxy_port="7890"
kioslaverc="/home/<UserName>/.config/kioslaverc"
firefox_profile="/home/<UserName>/.mozilla/firefox/xxx/user.js"

function proxy-set {
    export http_proxy=socks5://127.0.0.1:$proxy_port
    export https_proxy=socks5://127.0.0.1:$proxy_port
    echo -e "Proxy environment variable has been ${green}set${reset}."
}

function proxy-unset {
    unset http_proxy https_proxy
    echo -e "Proxy environment variable has been ${red}unset${reset}."
}

function kde-proxy-on {
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ftpProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "httpsProxy" "http://127.0.0.1 $proxy_port"
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "socksProxy" "socks://127.0.0.1 $proxy_port"
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 1
    echo -e "KDE proxy ${green}enabled${reset}."
}

function kde-proxy-off {
    kwriteconfig6 --file $kioslaverc --group "Proxy Settings" --key "ProxyType" 0
    echo -e "KDE proxy ${red}disabled${reset}."
}

function ff-proxy-on {
    cat <<EOF >"$firefox_profile"
user_pref("network.proxy.backup.ssl", "127.0.0.1");
user_pref("network.proxy.backup.ssl_port", $proxy_port);
user_pref("network.proxy.http", "127.0.0.1");
user_pref("network.proxy.http_port", $proxy_port);
user_pref("network.proxy.share_proxy_settings", true);
user_pref("network.proxy.socks", "127.0.0.1");
user_pref("network.proxy.socks_port", $proxy_port);
user_pref("network.proxy.socks_remote_dns", true);
user_pref("network.proxy.ssl", "127.0.0.1");
user_pref("network.proxy.ssl_port", $proxy_port);
user_pref("network.proxy.type", 1);
EOF
    echo -e "Firefox Proxy ${green}enabled${reset}. Restart Firefox to apply."
}

function ff-proxy-off {
    echo 'user_pref("network.proxy.type", 0);' >"$firefox_profile"
    echo -e "Firefox Proxy ${red}disabled${reset}. Restart Firefox to apply."
}

# 保持以下内容位于文件末尾
unset $reset
unset $black
unset $red
unset $green
unset $yellow
unset $blue
unset $purple
unset $cyan
unset $white

unset $proxy_port
unset $kioslaverc
unset $firefox_profile
```

然后在 `/etc/bash.bashrc` 中写入: 

```bash
if [ -f /etc/proxy-custom ]; then
  source /etc/proxy-custom
fi
```

### 当前配置重启

将以下代码添加到 `/etc/proxy-custom` 中: 

```bash
function sing-box-restart {
    dir_config="/etc/sing-box"
    service="sing-box.service"
    sing-box check -c $dir_config/config.json &>/dev/null
    if [ $? -ne 0 ]; then
        echo -e "File ${yellow}$dir_config/config.json${reset} is ${red}invalid${reset}."
        sleep 2
        exit 1
    fi
    sudo systemctl restart $service
    sleep 5
    sub_state=$(systemctl show -p SubState --value $service)
    if [ $sub_state == "running" ]; then
        echo -e "$service state: ${green}$sub_state${reset}."
        if grep -q '"type": "tun"' $dir_config/config.json; then
            echo -e "Use ${blue}tun${reset} mode."
            kde-proxy-off
            # ff-proxy-off
        else
            echo -e "Use ${blue}system proxy${reset} mode."
            kde-proxy-on
            # ff-proxy-on
        fi
    else
        echo -e "$service state: ${red}$sub_state${reset}."
    fi
}
```

### 停止运行

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function sing-box-stop {
    service="sing-box.service"
    sudo systemctl stop $service
    sleep 1
    sub_state=$(systemctl show -p SubState --value $service)
    echo -e "$service state: ${red}$sub_state${reset}."
}
```

### 切换模式重启

sing-box 的入站方式是固定在配置文件中的, 而且不支持通过 API 切换, 所以只能切换配置文件来切换 `mixed` 或 `tun` 模式。

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function sing-box-switch {
    dir_config="/etc/sing-box"
    service="sing-box.service"
    if grep -q '"type": "tun"' $dir_config/config.json; then
        inbound="mixed"
    else
        inbound="tun"
    fi
    sing-box check -c $dir_config/config_$inbound.json &>/dev/null
    if [ $? -ne 0 ]; then
        echo -e "File ${yellow}$dir_config/config_$inbound.json${reset} is ${red}invalid${reset}."
        sleep 2
        exit 1
    fi
    sudo cp $dir_config/config_$inbound.json $dir_config/config.json
    sudo systemctl restart $service
    sleep 5
    sub_state=$(systemctl show -p SubState --value $service)
    if [ $sub_state == "running" ]; then
        echo -e "$service state: ${green}$sub_state${reset}."
        if [ $inbound == "mixed" ]; then
            echo -e "Use ${blue}system proxy${reset} mode."
            kde-proxy-on
            # ff-proxy-on
        elif [ $inbound == "tun" ]; then
            echo -e "Use ${blue}tun${reset} mode."
            kde-proxy-off
            # ff-proxy-off
        fi
    else
        echo -e "$service state: ${red}$sub_state${reset}."
    fi
}
```

### 查看日志

查看日志: 

```bash
sudo journalctl -u sing-box.service -o cat -e
```

查看实时日志: 

```bash
sudo journalctl -u sing-box.service -o cat -f
```