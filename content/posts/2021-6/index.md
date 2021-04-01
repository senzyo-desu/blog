---
title: "Linux环境变量配置文件"
date: 2021-03-16T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,Linux]
featuredImagePreview: ""
summary: "加载顺序: `/etc/profile`>`/etc/bash.bashrc`>`~/.profile`=`~/.bash_profile`>`~/.bashrc`。"
---

`/etc/envirnoment` 与 shell 无关, 所以无法使用脚本或通配符展开。此文件仅接受 `variable=value` 键值对格式。

`/etc/profile` 仅初始化登录 shell 的变量。但它可以执行脚本 (比如 `/etc/profile.d/` 中的脚本) 并支持所有兼容 Bash 的 shell。

更改环境变量配置文件后, 手动 `source` 立刻生效, 比如: `source /etc/bash.bashrc`。

## 加载顺序

简单来说: `/etc/profile`>`/etc/bash.bashrc`>`~/.profile`=`~/.bash_profile`>`~/.bashrc`。

具体来说: 

- `/etc/profile`: 系统范围的配置文件, 对所有用户生效。
- `/etc/bash.bashrc`: 同样是系统范围的配置文件, 对所有用户生效。通常, `/etc/profile` 在登录时加载, 而 `/etc/bash.bashrc` 在每次启动新的 Bash shell 时加载。
- `~/.profile`: 用户的个人配置文件, 仅对当前用户生效。
- `~/.bash_profile`: 个人的 Bash 配置文件, 如果存在该文件, 它会覆盖 `~/.profile`。这个文件通常用于个性化配置, 并在登录时执行。
- `~/.bashrc`: 个人的 Bash 配置文件, 如果存在该文件, 它会在每次启动新的 Bash shell 时加载。这个文件通常包含一些定制的 Bash 设置和环境变量。

## 优先级

- `/etc/profile` 和 `/etc/bash.bashrc` 是系统级别的配置文件, 优先级较低。
- 用户级别的配置文件 `~/.profile`、`~/.bash_profile` 和 `~/.bashrc` 优先级较高, 个人配置会覆盖系统配置。

需要注意的是, 当用户登录时, 通常只会执行 `~/.bash_profile` 或 `~/.profile` 中的一个, 而不是两者都执行。如果两者都存在, 通常执行 `~/.bash_profile`。

--------------------

{{< admonition type=quote title="推荐阅读" open=true >}}
[Environment_variables](https://wiki.archlinux.org/title/Environment_variables)
{{< /admonition >}}
