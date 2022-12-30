---
title: "Fail2ban简单配置"
date: 2022-09-28T12:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Debian,Linux,Security]
featuredImagePreview: ""
summary: 在 Debian11 中搭配 Nftables 使用 Fail2ban。
---

## 设置fail2ban

安装`fail2ban`和`nftables`: 

```bash
apt install fail2ban nftables
```

设置为开机自启: 

```bash
systemctl enable fail2ban
systemctl enable nftables
```

启动: 

```bash
systemctl start fail2ban
systemctl start nftables
```

检查运行状态: 

```bash
systemctl status fail2ban
systemctl status nftables
```

编辑`fail2ban`的`jail`配置: 

```bash
cp /etc/fail2ban/jail.{conf,local}
vim /etc/fail2ban/jail.local
```

```
[INCLUDES]
before = paths-debian.conf

[DEFAULT]
bantime.rndtime = 30m
ignoreself = true
ignoreip = 127.0.0.1/8 ::1
ignorecommand =
bantime  = 10m
findtime  = 10m
maxretry = 5
maxmatches = %(maxretry)s
backend = auto
usedns = warn
logencoding = auto
enabled = false
mode = normal
filter = %(__name__)s[mode=%(mode)s]

# ACTIONS
destemail = root@localhost
sender = root@<fq-hostname>
mta = sendmail
protocol = tcp
chain = <known/chain>
port = 0:65535
fail2ban_agent = Fail2Ban/%(fail2ban_version)s

banaction = nftables-multiport
banaction_allports = nftables-allports
action_ = %(banaction)s[port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"]
action_mw = %(action_)s
            %(mta)s-whois[sender="%(sender)s", dest="%(destemail)s", protocol="%(protocol)s", chain="%(chain)s"]
action_mwl = %(action_)s
             %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]
action_xarf = %(action_)s
             xarf-login-attack[service=%(__name__)s, sender="%(sender)s", logpath="%(logpath)s", port="%(port)s"]
action_cf_mwl = cloudflare[cfuser="%(cfemail)s", cftoken="%(cfapikey)s"]
                %(mta)s-whois-lines[sender="%(sender)s", dest="%(destemail)s", logpath="%(logpath)s", chain="%(chain)s"]
action_blocklist_de  = blocklist_de[email="%(sender)s", service="%(__name__)s", apikey="%(blocklist_de_apikey)s", agent="%(fail2ban_agent)s"]
action_badips = badips.py[category="%(__name__)s", banaction="%(banaction)s", agent="%(fail2ban_agent)s"]
action_badips_report = badips[category="%(__name__)s", agent="%(fail2ban_agent)s"]
action_abuseipdb = abuseipdb
action = %(action_)s

# JAILS

[sshd]
enabled  = true
port     = 10022
filter   = sshd
mode     = aggressive
logpath  = %(sshd_log)s
backend  = %(sshd_backend)s
bantime  = 7d
findtime = 10m
maxretry = 2
```

重新启动`fail2ban `, 并检查其状态: 

```bash
systemctl restart fail2ban
systemctl status fail2ban
```

### fail2ban-client

使用`fail2ban-client`查看`fail2ban`的状态: 

```bash
fail2ban-client status sshd
```

封禁某一IP: 

```bash
fail2ban-client set sshd banip xx.xx.xx.xx
```

解封某一IP: 

```bash
fail2ban-client set sshd unbanip xx.xx.xx.xx
```

## 添加服务依赖

重新启动`nftables`时, 确保也重新启动`fail2ban`服务, 以便其添加防火墙规则, 为此需要在两者之间添加服务依赖关系, 创建一个新文件`/etc/systemd/system/fail2ban.service.d/override.conf`并填入以下内容: 

```
[Unit]
Requires=nftables.service
PartOf=nftables.service

[Install]
WantedBy=multi-user.target nftables.service
```

重新加载系统守护程序服务以包含此依赖项: 

```bash
systemctl daemon-reload
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. https://www.fail2ban.org/
2. https://wiki.nftables.org/
3. https://wiki.debian.org/nftables
{{< /admonition >}}
