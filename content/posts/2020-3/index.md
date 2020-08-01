---
title: "解除锐捷客户端对VMware限制"
date: 2020-06-01T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Hack,Network,Windows]
featuredImagePreview: "/2020-3/featuredImagePreview.svg"
summary: 锐捷客户端会每 30~60 秒就停止 VMware NAT Service 的运行, 这里通过修改 8021x.exe 来解除限制。
---

## VMware NAT Service停止运行原因

- 使用无线连接校园网时, 只需要在认证网页上登录即可, VMware NAT Service不受影响; 
- 而使用有线连接校园网时, 需要使用锐捷客户端来登录。

问题来了——客户端安装文件夹里的**8021.exe**程序大概每30~60秒就检查一次VMware NAT Service是否处于运行状态, 如果是, 就停止VMware NAT Service的运行, 所以使用NAT模式的VMware虚拟机就无法上网。

{{< image src="1.png" caption="锐捷客户端版本" height="auto" width="40%">}}

## 解决方法

{{< admonition type=success title="有效的方法" open=true >}}
替换代表`VMware NAT Service`的16进制编码, 使关闭VMware NAT Service的逻辑失去作用目标。
{{< /admonition >}}

1. 在任务管理器中先结束**RJSuService**, 然后再结束**锐捷认证客户端**。

{{< image src="2.png" caption="RJSuService服务进程" height="auto" width="100%">}}

{{< image src="3.png" caption="锐捷认证客户端进程" height="auto" width="100%">}}

2. 用[UltraEdit](https://www.ultraedit.com/downloads/ultraedit-download/)打开锐捷客户端安装文件夹里的8021.exe, 发现显示的数据是16进制的, 通过网上的**ASCII字符串转16进制工具**, 得知`VMware NAT Service`的16进制表示方式为`56 4d 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65`, 于是在UltraEdit中搜索, 发现只有一处。解决方法是用其他不存在的服务来替换它, 比如用`AAware NAT Service`的16进制表现方式`41 41 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65`来替换`VMware NAT Service`的16进制表示方式`56 4d 77 61 72 65 20 4e 41 54 20 53 65 72 76 69 63 65`, 如下图: 

{{< image src="4.png" caption="搜索VMware NAT Service位置" height="auto" width="80%">}}

{{< image src="5.png" caption="替换为AAware NAT Service" height="auto" width="80%">}}

{{< admonition type=warning title="警告" open=true >}}
不要改变字符长度, 比如改成`AAA Service`, 其16进制编码为`41 41 41 20 53 65 72 76 69 63 65`, 这样改动后, 此处16进制编码长度也会发生变化导致错误, 启动不了锐捷客户端。
{{< /admonition >}}

改好了以后, 如果没有失误, 就会发现系统服务中的VMware NAT Service不会被停止运行了。

{{< admonition type=failure title="网上其他失效的方法" open=true >}}
1. 更改8021x.exe对于VMware NAT Service的检测时间
2. 直接搜索`VMware NAT Service`, 替换为其他字符。
{{< /admonition >}}

## 虚拟机NAT模式联网

因为`VMware NAT Service`总是被锐捷客户端停止运行, 所以处理好锐捷客户端之后才弄明白, 只要`VMware NAT Service`、`VMware DHCP Service`处于运行状态, 不必在**主机**→**网络共享中心**→**更改适配器**中, 共享主机的网卡到虚拟机的网卡: 

{{< image src="6.png" caption="主机网卡共享给虚拟机" height="auto" width="50%">}}

VMware中的**编辑→虚拟网络编辑器**中, NAT模式保持默认设置, 不是默认就还原默认设置: 

{{< image src="7.png" caption="NAT设置" height="auto" width="50%">}}

**虚拟机→虚拟机设置→硬件→网络适配器**中选择NAT模式: 

{{< image src="8.png" caption="选择NAT模式" height="auto" width="50%">}}

**主机**→**网络共享中心**→**更改适配器**中, 虚拟机网卡选择自动获取IP和DNS即可, 到此为止, 虚拟机已经能上网了, 不必做更多的配置。