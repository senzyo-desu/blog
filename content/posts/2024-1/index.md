---
title: "Generate config of sing-box"
date: 2024-01-07T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,Proxy]
series: [Use sing-box]
series_weight: 1
featuredImagePreview: ""
summary: 请求订阅信息并根据模板生成用于 sing-box 的配置文件。
toc:
  enable: false
---

由于: 

- sing-box 不像 clash 那样支持 `proxy-providers`。
- sing-box 在 Linux 和 Windows 平台的核心程序不支持定时更新配置文件。
- 机场不支持 sing-box。

所以只能使用“(远端) 后端下载订阅→提取节点→根据模板生成适用的配置→客户端下载配置”的方式, 那么选用哪个后端呢? 

[serenity](https://github.com/SagerNet/serenity)、[subconverter](https://github.com/tindy2013/subconverter) 及其 [二次开发版本](https://github.com/asdlokj1qpi23/subconverter) 都还行, 但我选择更简便的 [sing-box-subscribe](https://github.com/Toperlock/sing-box-subscribe)。

1. Fork [sing-box-subscribe](https://github.com/Toperlock/sing-box-subscribe)。
2. 默认的 `up_mbps` 和 `down_mbps` 可能不适合自己的网络, 建议自行对带宽测速后, 更改 [hysteria.py](https://github.com/Toperlock/sing-box-subscribe/blob/main/parsers/hysteria.py) 和 [hysteria2.py](https://github.com/Toperlock/sing-box-subscribe/blob/main/parsers/hysteria2.py) 中的这两个值。
3. 在 [Vercel](https://vercel.com/new) 部署 `sing-box-subscribe`。
4. 创建用于存放自定义模板的 GitHub 仓库, 比如 [sing-box-templates](https://github.com/senzyo/sing-box-templates)。
5. 拼接出最终的 URL, 比如: 
   
    ```bash
    #!/bin/bash
    
    url_gene="https://a.com"  # 生成配置的后端地址
    url_sub="https://b.com"   # 来自机场的订阅链接
    url_tpl="https://raw.githubusercontent.com/senzyo/sing-box-templates/normal/tun/dot/8.8.8.8/ghproxy.net/config.json"  # 配置所用模板的地址
    url_dl="$url_gene/config/$url_sub&ua=clashmeta&emoji=1&file=$url_tpl"
    echo $url_dl
    # curl -L -o config.json "$url_dl"
    ```

6. 在 Android 或 Apple 设备的 sing-box 图形客户端中添加这个最终的 URL 作为订阅链接。
7. 对于 Linux 和 Windows, 阅读 [sing-box on Linux](https://senzyo.net/2024-2/#日常使用) 和 [sing-box on Windows](https://senzyo.net/2024-3/#日常使用)。


