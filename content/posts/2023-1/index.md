---
title: "Aria2和AriaNg"
date: 2023-03-12T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Browser,Download]
featuredImagePreview: ""
summary: Aria2 配置, AriaNg 以及 Chromium 系浏览器扩展。
---

## Aria2

Windows 需要手动从 [GitHub](https://github.com/aria2/aria2/releases/latest) 下载 aria2。

Arch 可以直接 `sudo pacman -S aria2`。

## 配置文件

1. 下载 [aria2.conf](https://github.com/P3TERX/aria2.conf/blob/master/aria2.conf), 参考其注释进行修改。
2. 在 `aria2.conf` 中的 `bt-tracker=` 后添加 [Tracker](https://github.com/XIU2/TrackersListCollection) 地址, 建议使用 [all_aria2.txt](https://ghproxy.net/https://raw.githubusercontent.com/XIU2/TrackersListCollection/master/all_aria2.txt)。
3. 按照配置文件, 创建需要的空文件 `aria2.session`, 下载 [dht.dat](https://ghproxy.net/https://raw.githubusercontent.com/XIU2/TrackersListCollection/master/dht.dat) 和 [dht6.dat](https://ghproxy.net/https://raw.githubusercontent.com/XIU2/TrackersListCollection/master/dht.dat)。
4. 修改 `aria2.conf` 中, 以上各文件的引用路径。
5. Windows 中, 我一般将 aria2 本体和配置等文件放在一起, 比如 `C:\Users\<UserName>\Apps\Aria2`。
6. Linux 中, 通过 pacman 安装的 aria2, 其路径是 `/usr/bin/aria2c`。配置等文件一般放在 `$XDG_CONFIG_HOME/aria2/` 中, 比如 `~/.config/aria2/`。

## Windows开机自启动

### 方案一: 任务计划程序

打开任务计划程序, 点击右侧操作列表中的“创建任务”:

1. 常规

   `更改用户或组`, 输入 `system`, 点击 `检查名称` 按钮, 变为 <u>SYSTEM</u>, 即使用 `SYSTEM` 账户。

    > 如果要使用 `falloc` 的文件预分配方式, 又出现报错 `[WARN] Gaining privilege SeManageVolumePrivilege failed`, 则需要勾选 `使用最高权限运行`。

2. 触发器

    新建, `开始任务` 选择 `登录时` 或 `启动时`, 按需选择 `延迟任务时间`, 确认勾选 `已启用`。

3. 操作

    新建, `操作` 选择 `启动程序`, `程序或脚本` 填写 `aria2c.exe`, `添加参数` 填写 `--conf=aria2.conf`, `起始于` 填写 Aria2 工作目录。

4. 条件

    全部取消勾选, 包括灰显的。

5. 设置

   1. 仅勾选 `允许按需运行任务`。
   2. 最下方的 `如果此任务已经运行, 以下规则适用`, 确认选择 `请勿启动新实例`。

6. 运行任务, 检查是否正常。

### 方案二: Windows服务

利用 [winsw](https://github.com/winsw/winsw/releases) 安装 Windows 服务。

1. 将以下内容保存为 `Aria2 Service.xml`, 和 `winsw.exe` 一起放入 Aria2 工作目录中。

    ```xml
    <service>
        <startmode>Automatic</startmode>
        <id>Aria2</id>
        <name>Aria2</name>
        <description></description>
        <executable>aria2c.exe</executable>
        <arguments>--conf=aria2.conf</arguments>
        <log mode="none"></log>
    </service>
    ```

2. 在 Aria2 工作目录中运行以下命令将 Aria2 安装为 Windows 服务: 

    ```shell
    winsw.exe install "Aria2 Service.xml"
    ```

3. 运行服务, 检查是否正常。

## Linux开机自启动

为 aria2 设置守护进程: 

```bash
vim ~/.config/systemd/user/aria2.service
```

写入以下内容:

```
[Unit]
Description=aria2 service
Documentation=https://wiki.archlinux.org/title/Aria2
After=network.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/bin/aria2c --conf-path=/home/<UserName>/.config/aria2/aria2.conf --daemon
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=10s
LimitNOFILE=infinity

[Install]
WantedBy=default.target
```

重载 systemd: 

```bash
systemctl --user daemon-reload
```

设置开机自启并立刻启动: 

```bash
systemctl --user enable --now aria2.service
```

查看状态: 

```bash
systemctl --user status aria2.service
```

更多 Systemd 信息参考 [Systemd/User](https://wiki.archlinux.org/title/Systemd/User) 和 [Systemd](https://wiki.archlinux.org/title/Systemd)。

## AriaNg

配置 RPC 连接地址和密钥, 连接成功后就可通过 AriaNg 页面来管理 aria2。

### Chromium系

安装扩展 [Aria2 for Chrome](https://chrome.google.com/webstore/detail/aria2-for-chrome/mpkodccbngfoacfalldjimigbofkhgjn) 或 [Aria2 for Edge](https://microsoftedge.microsoft.com/addons/detail/aria2-for-edge/jjfgljkjddpcpfapejfkelkbjbehagbh), 这两个扩展是相同的。

这个扩展可以拦截浏览器的下载请求, 转为使用 aria2 下载。并且集成了 [AriaNg](http://ariang.mayswind.net/zh_Hans/), 点击扩展就可以打开 AriaNg。

### Firefox

很遗憾上面提到的扩展没有 Firefox 版。

下载 AriaNg 的 [本地网页](https://github.com/mayswind/AriaNg/releases/latest), 放入 aria2 的配置文件所在目录, 用浏览器访问 AriaNg 的 HTML 文件路径即可打开 AriaNg。
