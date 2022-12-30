---
title: "Nftables简单配置"
date: 2022-10-01T12:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Debian,Linux,Nftables]
featuredImagePreview: ""
summary: 简单配置服务器的 Nftables, Input 只允许 `SSH`、`HTTP` 和 `HTTPS` 通行, Output 全部允许, Forward 全部拒绝。
---

## 配置

虽然可以按部就班的敲命令, 但是我觉得直接编辑nftables的默认配置文件更快一些, 而且因为`nftables.service`启动时会自动加载默认配置文件, 这样相当于使规则持久化, 不用担心`nftables.service`停止时清空规则导致规则丢失。

### nftables默认配置文件

nftables默认配置文件一般是`/etc/nftables.conf`, 但是还是确定一下情况比较放心, 毕竟考虑到系统不同或版本不同, 默认配置文件还有可能是`/etc/nftables.start.conf`, 或者其他。

参考nftables官方文档[Advanced ruleset for dynamic environments](https://wiki.nftables.org/wiki-nftables/index.php/Advanced_ruleset_for_dynamic_environments), nftables默认配置文件的路径可以通过查看`nftables.service`文件得知, 但使用`find /etc -name "nftables.service"`搜索后发现我的Debian 11中`nftables.service`文件的路径并不是`/etc/systemd/system/nftables.service`, 而是`/etc/systemd/system/sysinit.target.wants/nftables.service`, 文件内容中注意到以下三行: 

```
...
ExecStart=/usr/sbin/nft -f /etc/nftables.conf
ExecReload=/usr/sbin/nft -f /etc/nftables.conf
ExecStop=/usr/sbin/nft flush ruleset
...
```

即`nftables.service`启动和重载时都加载`/etc/nftables.conf`文件, 停止时清空规则。

## 简单配置参考

参考nftables官方文档[Simple ruleset for a server](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server), 将`/etc/nftables.conf`文件更改为以下内容: 

```
#!/usr/sbin/nft -f

flush ruleset

table inet firewall {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state vmap { established : accept, related : accept, invalid : drop }
        iifname lo accept
        tcp dport { 22, 80, 443 } accept
        meta protocol vmap { ip : jump input_ipv4, ip6 : jump input_ipv6 }
        # https://wiki.nftables.org/wiki-nftables/index.php/Rejecting_traffic
        reject with icmpx type admin-prohibited
    }
    chain input_ipv4 {
        # icmp type echo-request limit rate 5/second accept
    }
    chain input_ipv6 {
        # icmpv6 type echo-request limit rate 5/second accept
    }
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    # no need to define output chain, default policy is accept if undefined.
}
```

重启`nftables.service`服务: 

```bash
systemctl restart nftables.service
```

检查nftables当前应用的规则: 

```bash
nft list ruleset
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [Simple ruleset for a server](https://wiki.nftables.org/wiki-nftables/index.php/Simple_ruleset_for_a_server)
2. [Nftables](https://wiki.archlinux.org/title/Nftables)
3. [nftables 配置与使用记录](https://blog.starryvoid.com/archives/1045.html)
4. [nftables 简明教程](https://www.hi-linux.com/posts/29206.html)
{{< /admonition >}}
