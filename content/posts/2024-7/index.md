---
title: "公共STUN"
date: 2024-05-02T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Network]
featuredImagePreview: ""
summary: " "
toc:
  enable: false
---

{{< admonition type=quote title="引用源" open=true >}}
https://github.com/heiher/natmap/issues/18#issue-1580804352
{{< /admonition >}}

## TCP/UDP

| Primary address          | Primary port | Alternate port | Notes |
| ------------------------ | ------------ | -------------- | ----- |
| turn.cloudflare.com      | 3478         | 53/udp,80/tcp  |       |
| fwa.lifesizecloud.com    | 3478         |                |       |
| stun.isp.net.au          | 3478         |                |       |
| stun.freeswitch.org      | 3478         |                |       |
| stun.voip.blackberry.com | 3478         |                |       |
| stun.nextcloud.com       | 3478         |                |       |
| stun.sipnet.com          | 3478         |                |       |
| stun.radiojar.com        | 3478         |                |       |
| stun.sonetel.com         | 3478         |                |       |
| stun.voipgate.com        | 3478         |                |       |

## UDP Only

| Primary address        | Primary port | Alternate port | Notes       |
| ---------------------- | ------------ | -------------- | ----------- |
| stun.miwifi.com        | 3478         |                |             |
| stun.chat.bilibili.com | 3478         |                |             |
| stun.cloudflare.com    | 3478         | 53             |             |
| stun.qq.com            | 3478         |                | Unavailable |

{{< admonition type=example title="更多" open=true >}}
更多 STUN server 参见 https://gist.github.com/mondain/b0ec1cf5f60ae726202e#file-public-stun-list-txt
{{< /admonition >}}
