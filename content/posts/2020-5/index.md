---
title: "Speedtest CLI使用帮助"
date: 2020-06-20T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Help,Network]
featuredImagePreview: "/2020-5/featuredImagePreview.svg"
summary: Speedtest CLI 的命令使用帮助。
---

## 下载地址

来自`speedtest.net`的[Speedtest Cli](https://www.speedtest.net/zh-Hans/apps/cli), **不是**`speedtest.cn`的[命令行测速工具](https://b.speedtest.cn/cli)。

## 简介
```
speedtest 
    [-aAbBfhiIpPsv] [--ca-certificate=path] 
    [--format=[=format-type]] [--help] 
    [--interface=interface] [--ip=ip_address] 
    [--output-header] [--precision=num_decimal_places] 
    [--progress=yes|no] [--selection-details] 
    [--server-id=id] [--servers] 
    [--unit=[=unit-of-measure]] [--version]
```

## 选项
| **-h, --help**                                   | 打印使用信息                                       |
| ------------------------------------------------ | -------------------------------------------------- |
| **-v**                                           | 记录详细程度, 为更高的详细程度指定多次 (例如-vvv)  |
| **-V, --version**                                | 打印版本号                                         |
| **-L, --servers**                                | 列出最近的服务器                                   |
| **--selection-details**                          | 显示服务器选择详细信息                             |
| **-s**  *id*, **--server-id**=*id*               | 使用服务器id从服务器列表中指定服务器               |
| **-o** *hostname*, **--host**=*hostname*         | 从服务器列表中使用其主机名指定服务器               |
| **-f**  *format_type*,**--format**=*format_type* | 输出格式 (默认值=**human-readable**)               |

{{< admonition type=info title="注意" open=true >}}
Machine readable (csv、tsv、json、json、json pretty) 使用字节作为最大精度的测量单位。
{{< /admonition >}}

*format_type*的值如下: 

| format             | type                            |
| ------------------ | :------------------------------ |
| **human-readable** | 人类可读输出                    |
| **csv**            | 逗号分隔值                      |
| **tsv**            | 制表符分隔值                    |
| **json**           | javascript对象表示法 (compact)  |
| **jsonl**          | javascript对象表示法 (lines)    |
| **json-pretty**    | javascript对象表示法 (pretty)   |

* `--output-header`显示CSV和TSV格式的输出标题

* `-u*` unit_of_measure, `--unit*` unit_of_measure
  
  用于显示速度的输出装置 (注: 这仅适用于对于`human-readable`输出格式, 默认单位为Mbps) 

  | 单位                   | 解释                        |
  | ---------------------- | --------------------------- |
  | **bps**                | bits 每秒 (十进制前缀)      |
  | **kbps**               | kilobits 每秒 (十进制前缀)  |
  | **Mbps**               | megabits 每秒 (十进制前缀)  |
  | **Gbps**               | gigabits 每秒 (十进制前缀)  |
  | **kibps**              | kilobits 每秒 (二进制前缀)  |
  | **Mibps**              | megabits 每秒 (二进制前缀) |
  | **Gibps**              | gigabits 每秒 (二进制前缀) |
  | **B/s**                | bytes 每秒                  |
  | **kB/s**               | kilobytes 每秒              |
  | **MB/s**               | megabytes 每秒              |
  | **GiB/s**              | gigabytes 每秒              |
  | **auto-binary-bytes**  | 自动二进制字节              |
  | **auto-decimal-bytes** | 自动十进制字节              |
  | **auto-binary-bits**   | 自动二进制位                |
  | **auto-decimal-bits**  | 自动十进制位                |
  
* `-a`  
  
  [**-u auto-decimal-bits**] 的快捷方式
  
* `-A`  
  
   [**-u auto-decimal-bytes**] 的快捷方式
  
* `-b`  
  
   [**-u auto-binary-bits**] 的快捷方式
  
* `-B`  
  
   [**-u auto-binary-bytes**] 的快捷方式
  
* `-P` *decimal_places* `--precision`=*decimal_places*  

   要使用的小数位数 (默认 = 2, 有效 = 0~8)

* `-p` *yes* | *no* `--progress`=*yes* | *no*  

   启用或禁用进度条 (默认 = yes, 互动时)

* `-I` *interface* `--interface`=*interface*

   连接到服务器时尝试绑定到指定的接口

* `-i` *ip_address* `--ip`=*ip_address*

   连接到服务器时尝试绑定到指定的IP地址

* `--ca-certificate`=*path*

  CA证书捆绑包的路径, 请参阅下面的注释。

## 输出
成功执行后, 应用程序将退出, 退出代码为0。结果将包括延迟、抖动、下载、上载、数据包丢失 (如果可用) 和结果URL。

延迟和抖动将以毫秒表示。下载和上传单元将取决于输出格式以及是否指定了单位。

**human-readable**格式默认为Mbps和任何机器可读格式 (csv、tsv、json、json、json pretty) 使用字节作为最大精度的度量单位。

数据包丢失用百分比表示, 当数据包丢失在执行的网络环境中不可用时, **不可用**。

结果URL可用于共享您的结果, 将 **.png**附加到结果URL将创建一个可共享的结果图像。

**human-readable 结果示例**

```shell
$ speedtest
    Speedtest by Ookla

     Server: Speedtest.net - New York, NY (id = 10390)
        ISP: Comcast Cable
    Latency:    57.81 ms   (3.65 ms jitter)
   Download:    76.82 Mbps (data used: 80.9 MB)
     Upload:    37.58 Mbps (data used: 65.3 MB)
Packet Loss:     0.0%
 Result URL: https://www.speedtest.net/result/c/8ae1200c-e639-45e5-8b55-41421a079250
```

## 网络超时值
默认情况下, 网络请求将超时设置为**10**秒。唯一的例外是延迟测试, 它将超时设置为**15**秒。

## 致命错误
一旦出现致命错误, 应用程序将以非零退出代码退出。
**初始化致命错误示例: **

`Configuration - Couldn't connect to server (Network is unreachable)`

`Configuration - Could not retrieve or read configuration (ConfigurationError)`

**阶段执行致命错误示例: **

`[error] Error: [1] Latency test failed for HTTP`

`[error] Error: [36] Cannot open socket: Operation now in progress`

`[error] Failed to resolve host name. Cancelling test suite`

`[error] Host resolve failed: Exec format error`

`[error] Cannot open socket: No route to host`

`[error] Server Selection - Failed to find a working test server. (No Servers)`

## SSL证书位置

默认情况下, 会检查linux计算机上CA证书捆绑包的以下路径: 

```bash
/etc/ssl/certs/ca-certificates.crt
/etc/pki/tls/certs/ca-bundle.crt
/usr/share/ssl/certs/ca-bundle.crt
/usr/local/share/certs/ca-root-nss.crt
/etc/ssl/cert.pem
```

如果测试中的设备**没有**以上提到的文件之一, 那么curl项目提供的规范和最新的CA证书包可以手动完成下载到特定位置。可根据以下示例将此特定位置作为参数提供: 

```bash
wget https://curl.haxx.se/ca/cacert.pem
./ookla --ca-certificate=./cacert.pem
```
