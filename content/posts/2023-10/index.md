---
title: "Clash on Linux"
date: 2023-12-17T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Proxy,Linux]
series: [Use Clash Core]
series_weight: 2
featuredImagePreview: ""
summary: 本文不使用 Clash Verge 等 GUI 客户端, 而是通过 systemd 服务和 Dashboard 网页来控制 Clash Meta Core (mihomo)。
---

{{< admonition type=info title="注意" open=true >}}
此文仅适用于客户端, 不适用于服务器和路由器。
{{< /admonition >}}

{{< admonition type=tip title="改名" open=true >}}
`Clash Meta` 改名为 `mihomo`。
{{< /admonition >}}

## Core

参考 [FAQ](https://wiki.metacubex.one/faq/faq/) 下载 [二进制文件](https://wiki.metacubex.one/startup/), 解压后将其重命名为 `mihomo`。

```bash
chmod +x mihomo
sudo mv mihomo /usr/local/bin/
```

## 配置文件

参考 [Clash配置文件](../2023-7/#配置文件)。

```bash
mkdir /etc/mihomo
sudo cp config.yaml /etc/mihomo/
```

## Dashboard

下载 [Yacd-meta](https://ghproxy.net/https://github.com/MetaCubeX/Yacd-meta/archive/refs/heads/gh-pages.zip) 或者 [MetaCubeXD](https://ghproxy.net/https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip), 解压后将文件夹重命名为 `ui`。

```bash
sudo cp -r ui /etc/mihomo/
```

## 设置守护进程

参考 [创建运行服务](https://wiki.metacubex.one/startup/service/)。

```bash
sudo vim /etc/systemd/system/mihomo.service
```

写入以下内容:

```
[Unit]
Description=mihomo service
Documentation=https://wiki.metacubex.one
After=network.target NetworkManager.service systemd-networkd.service

[Service]
Type=simple
LimitNPROC=500
LimitNOFILE=1000000
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH CAP_DAC_OVERRIDE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE CAP_SYS_TIME CAP_SYS_PTRACE CAP_DAC_READ_SEARCH CAP_DAC_OVERRIDE
Restart=on-failure
RestartSec=10s
ExecStartPre=/usr/bin/sleep 1s
ExecStart=/usr/local/bin/mihomo -d /etc/mihomo
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

重载 systemd: 

```bash
sudo systemctl daemon-reload
```

{{< admonition type=success title="完成了" open=true >}}
现在可以运行 `sudo systemctl start mihomo.service` 测试一下整个流程, 应该一切正常。
{{< /admonition >}}

## 日常使用

### 开关系统和浏览器代理

Chrome 会跟随 KDE 系统代理, Firefox 则不然。对于 KDE 系统代理和 [Firefox](../2024-4) 代理, 参考 [开关系统和浏览器代理](../2024-2/#开关系统和浏览器代理)。

### 当前配置重启

参考 [开关系统和浏览器代理](../2024-2/#开关系统和浏览器代理), 将以下代码添加到 `/etc/proxy-custom` 中: 

```bash
function mihomo-restart {
    dir_config="/etc/mihomo"
    service="mihomo.service"
    sudo systemctl restart $service
    sleep 3
    sub_state=$(systemctl show -p SubState --value $service)
    if [ $sub_state == "running" ]; then
        echo -e "$service state: ${green}$sub_state${reset}."
        secret=$(grep secret $dir_config/config.yaml | cut -d ' ' -f 2)
        controller_api=$(grep external-controller $dir_config/config.yaml | cut -d ' ' -f 2)
        response=$(curl -s -H "Authorization: Bearer ${secret}" -H "Content-Type: application/json" -X GET "http://${controller_api}/configs")
        tun_state=$(echo "$response" | jq -r '.tun.enable')
        if [ "$tun_state" == "true" ]; then
            echo -e "Tun ${green}enabled${reset}."
            echo -e "Use ${blue}tun${reset} mode."
            kde-proxy-off
            # ff-proxy-off
        else
            echo -e "Tun ${red}disabled${reset}."
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
function mihomo-stop {
    service="mihomo.service"
    sudo systemctl stop $service
    sleep 1
    sub_state=$(systemctl show -p SubState --value $service)
    echo -e "$service state: ${red}$sub_state${reset}."
}
```

### 切换模式重启

将以下代码追加到 `/etc/proxy-custom` 中: 

```bash
function mihomo-switch {
    dir_config="/etc/mihomo"
    service="mihomo.service"
    sub_state=$(systemctl show -p SubState --value $service)
    if [ $sub_state != "running" ]; then
        echo -e "$service state: ${red}$sub_state${reset}."
        sudo systemctl restart $service
        sleep 3
        sub_state=$(systemctl show -p SubState --value $service)
        if [ $sub_state != "running" ]; then
            echo -e "$service state: ${red}$sub_state${reset}."
            exit 1
        fi
    fi
    if [ $sub_state == "running" ]; then
        echo -e "$service state: ${green}$sub_state${reset}."
        secret=$(grep secret $dir_config/config.yaml | cut -d ' ' -f 2)
        controller_api=$(grep external-controller $dir_config/config.yaml | cut -d ' ' -f 2)
        response=$(curl -s -H "Authorization: Bearer ${secret}" -H "Content-Type: application/json" -X GET "http://${controller_api}/configs")
        tun_state=$(echo "$response" | jq -r '.tun.enable')
        if [ "$tun_state" == "true" ]; then
            curl -H "Authorization: Bearer ${secret}" -H "Content-Type: application/json" -X PATCH -d '{"tun":{"enable":false}}' "http://${controller_api}/configs"
            echo -e "Tun ${red}disabled${reset}."
            echo -e "Use ${blue}system proxy${reset} mode."
            kde-proxy-on
            # ff-proxy-on
        else
            curl -H "Authorization: Bearer ${secret}" -H "Content-Type: application/json" -X PATCH -d '{"tun":{"enable":true}}' "http://${controller_api}/configs"
            echo -e "Tun ${green}enabled${reset}."
            echo -e "Use ${blue}tun${reset} mode."
            kde-proxy-off
            # ff-proxy-off
        fi
    fi
}
```

### 查看日志

查看日志: 

```bash
sudo journalctl -u mihomo.service -o cat -e
```

查看实时日志: 

```bash
sudo journalctl -u mihomo.service -o cat -f
```
