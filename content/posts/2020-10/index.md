---
title: "APT保持指定的软件包不更新"
date: 2020-07-14T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [APT,Debian]
featuredImagePreview: ""
summary: 使用 `apt-mark` 或 `dpkg --set-selections` 或 `aptitude` 保持指定的软件包不更新。
---

## apt-mark

检查可更新软件包: 

```bash
sudo apt update
sudo apt list --upgradable
```

将`hold`选项传递给`apt mark`命令, 将软件包标记为保留, 这将阻止软件包自动安装、升级或删除: 

```bash
sudo apt-mark hold <package-name1> <package-name2> ...
```

显示所有保留状态的软件包: 

```bash
apt-mark showhold
```

取消软件包的保留状态: 

```bash
sudo apt-mark unhold <package-name1> <package-name2> ...
```

## dpkg

将软件包标记为保留: 

```bash
echo "<package-name> hold" | sudo dpkg --set-selections
```

检查指定包的保留状态: 

```bash
dpkg --get-selections | grep <package-name>
```

取消软件包的保留状态: 

```bash
echo "<package-name> install" | sudo dpkg --set-selections
```

## aptitude

将软件包标记为保留: 

```bash
sudo aptitude hold <package-name>
```

取消软件包的保留状态: 

```bash
sudo aptitude unhold <package-name>
```

还可以阻止软件包升级到特定版本, 同时允许自动升级到未来版本, 即跳过某一版本的更新: 

```bash
sudo aptitude forbid-version <package-name>=<version>
```

比如: 

```bash
sudo aptitude forbid-version bash=5.0-6ubuntu1.1
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
[apt-get hold back packages on Ubuntu / Debian Linux](https://www.cyberciti.biz/faq/apt-get-hold-back-packages-command/)
{{< /admonition >}}