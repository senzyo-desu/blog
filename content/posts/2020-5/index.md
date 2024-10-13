---
title: "Speedtest CLI使用帮助"
date: 2020-06-20T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Help,Network]
featuredImagePreview: ""
summary: 翻译 Speedtest CLI 的使用帮助文档。
---

{{< admonition type=info title="注意" open=true >}}
本文是 Speedtest CLI 使用帮助文档的翻译。

本文中的 `speedtest` 是指 `Ookla` 的 [speedtest.net](https://www.speedtest.net/zh-Hans/apps/cli), 不是 `巨擎网络科技` 的 [speedtest.cn](https://b.speedtest.cn/cli)。
{{< /admonition >}}


## 概要

`speedtest` [`--help`] [`-aAbBfhiIpPsv`] [`--ca-certificate=path`] [`--format`=\<format-type>]
[`--interface=interface`] [`--ip=ip_address`] [`--output-header`] [`--precision`=\<num_decimal_places>]
[`--progress`=\<yes|no>] [`--selection-details`] [`--server-id`=\<id>] [`--servers`] [`--unit`=\<unit-of-measure>] [`--version`]

## 描述

`speedtest` 是一款测量客户端与附近 Speedtest 服务器之间网络连接的延迟、抖动、丢包率、下载带宽和上传带宽的应用程序。

## 选项

- `-h`, `--help`:
  打印使用信息
- `-v`:
  日志详细级别, 多次指定以提高详细级别 (例如 `-vvv`) 
- `-V`, `--version`:
  打印版本号
- `-L`, `--servers`:
  列出最近的服务器
- `--selection-details`:
  显示服务器选择详细信息
- `-s` *id*, `--server-id`=*\<id>*:
  使用其 ID 从服务器列表中指定服务器
- `-o` *hostname*, `--host`=*\<hostname>*:
  使用其主机名从服务器列表中指定服务器
- `-f` *\<format-type>*, `--format`=*\<format-type>*:
  输出格式 (默认值为 `human-readable`)。有关详细信息, 请参阅下面的 [输出格式](#输出格式)。
 - `--progress-update-interval`=*\<interval>*:
  进度更新间隔 (100-1000 毫秒) 
- `--output-header`:
  显示 CSV 和 TSV 格式的输出标题
- `-u` *\<unit_of_measure>*, `--unit`=*\<unit_of_measure>*:
  使用 `human-readable` 输出格式时显示速度的输出单位。默认单位为 `Mbps`。有关更多详细信息, 请参阅 [度量单位](#度量单位)。
- `-a`:
  [`-u` *\<auto-decimal-bits>*] 的快捷方式
- `-A`:
  [`-u` *\<auto-decimal-bytes>*] 的快捷方式
- `-b`:
  [`-u` *\<auto-binary-bits>*] 的快捷方式
- `-B`:
  [`-u` *\<auto-binary-bytes>*] 的快捷方式
- `-P` *\<decimal_places>*, `--precision`=*\<decimal_places>*:
  要使用的小数位数 (默认值 = 2, 有效值 = 0-8)。仅适用于 `human-readable` 输出格式。
- `-p` *\<yes>|\<no>*, `--progress`=*\<yes>|\<no>*:
  启用或禁用进度条 (默认值为 `yes`, 在交互模式下) 
- `-I` *\<interface>*, `--interface`=*\<interface>*:
  尝试在连接到服务器时绑定到指定的接口
- `-i` *\<ip_address>*, `--ip`=*\<ip_address>*:
  尝试在连接到服务器时绑定到指定的 IP 地址
- `--ca-certificate`=*\<path>*:
  CA 证书束的路径, 请参阅下面的 [SSL 证书位置](#ssl-证书位置)。

## 输出格式

这些是 Speedtest CLI 可用的输出格式, 使用 `-f` 或 `--format` 标志指定。所有机器可读格式 (`csv`, `tsv`, `json`, `jsonl` 和 `json-pretty`) 都使用 **字节** 表示数据大小, 使用 **字节每秒** 表示速度, 使用 **毫秒** 表示持续时间。它们还始终使用最大精度输出。

- `human-readable`:
  人类可读的输出
- `csv`:
  逗号分隔值
- `tsv`:
  制表符分隔值
- `json`:
  JavaScript 对象表示法 (紧凑型) 
- `jsonl`:
  JavaScript 对象表示法 (行式) 
- `json-pretty`:
  JavaScript 对象表示法 (美化格式) 

## 度量单位

对于人类可读的输出格式, 您可以指定要使用的度量单位。默认单位为 `Mbps`。支持的单位如下所示。 

这些单位不适用于机器可读输出格式 (`json`, `jsonl`, `csv` 和 `tsv`)。

### 十进制选项

1000 的倍数。

- `bps`:
  比特 (bits) 每秒
- `kbps`:
  千比特 (kilobits) 每秒
- `Mbps`:
  兆比特 (megabits) 每秒
- `Gbps`:
  吉比特 (gigabits) 每秒
- `B/s`:
  字节 (bytes) 每秒
- `kB/s`:
  千字节 (kilobytes) 每秒
- `MB/s`:
  兆字节 (megabytes) 每秒
- `GB/s`:
  吉字节 (gigabytes) 每秒

### 二进制选项

1024 的倍数。

- `kibps`:
  千比特 (kibibits) 每秒
- `Mibps`:
  兆比特 (mebibits) 每秒
- `Gibps`:
  吉比特 (gibibits) 每秒
- `kiB/s`:
  千字节 (kibibytes) 每秒
- `MiB/s`:
  兆字节 (mebibytes) 每秒
- `GiB/s`:
  吉字节 (gibibytes) 每秒

### 自动缩放选项

自动单位将根据测量的速度缩放前缀。 

- `auto-decimal-bits`:
  十进制比特自动缩放
- `auto-decimal-bytes`:
  十进制字节自动缩放
- `auto-binary-bits`:
  二进制比特自动缩放
- `auto-binary-bytes`:
  二进制字节自动缩放

## 输出

成功执行后, 应用程序将退出, 退出代码为 `0`。结果将包括 **延迟**、**抖动**、**下载**、**上传**、**丢包率** (如果可用) 和 **结果 URL**。

延迟和抖动将以毫秒表示。下载和上传单位将取决于输出格式以及是否指定了单位。人类可读格式默认为 `Mbps`, 任何机器可读格式 (`csv`, `tsv`, `json`, `jsonl` 和 `json-pretty`) 都使用 `bytes` 作为度量单位, 并使用最大精度。丢包率以百分比表示, 如果在执行网络环境中不可用, 则表示为 `不可用`。

每秒字节数的测量值可以通过将 `每秒字节数的值` 除以 `125,000` 来转换为人类可读输出格式的默认单位 `Mbps`。例如: 

$38404104 \text{ bytes/s (B/s)} = \frac{38404104}{125} \text{ kilobits/s (kbps)} = 307232.832 \text{ kilobits/s (kbps)} = \frac{307232.832}{1000} \text{ megabits/s (Mbps)} = 307.232832 \text{ megabits/s (Mbps)}$

值 `125` 源自 `1000 / 8`: $\frac{byte}{kilobit} = \frac{8\ bits}{1000\ bits} = \frac{1}{125}$, 即 1 `kilobit (kb)` = 125 `bytes (B)`

结果 URL 可用于共享您的结果, 在结果 URL 后追加 `.png` 将创建一个可共享的结果图像。

**人类可读结果示例:**

```bash
$ speedtest
   Speedtest by Ookla

      Server: SUNET - Stockholm (id: 26852)
         ISP: Bahnhof AB
Idle Latency:     5.04 ms   (jitter: 0.04ms, low: 5.01ms, high: 5.07ms)
    Download:   968.73 Mbps (data used: 117.5 MB)                                                   
                 12.10 ms   (jitter: 1.71ms, low: 6.71ms, high: 18.82ms)
      Upload:   942.13 Mbps (data used: 114.8 MB)                                                   
                  9.94 ms   (jitter: 1.10ms, low: 5.30ms, high: 12.72ms)
 Packet Loss:     0.0%
  Result URL: https://www.speedtest.net/result/c/d1c46724-50a3-4a59-87ca-ffc09ea014b2
```

## 网络超时值

默认情况下, 网络请求设置的超时时间为 `10` 秒。唯一的例外是延迟测试, 它设置的超时时间为 `15` 秒。

## 致命错误

发生致命错误时, 应用程序将退出并返回非零退出代码。

`初始化致命错误示例:`

```
Configuration - Couldn't connect to server (Network is unreachable)
配置 - 无法连接到服务器 (网络不可达)
```

```
Configuration - Could not retrieve or read configuration (ConfigurationError)
配置 - 无法检索或读取配置 (ConfigurationError)
```

`阶段执行致命错误示例:`

```
[error] Error: [1] Latency test failed for HTTP.
[错误] 错误: [1] HTTP 延迟测试失败。
```

```
[error] Error: [36] Cannot open socket: Operation now in progress.
[错误] 错误: [36] 无法打开套接字: 操作正在进行中。
```

```
[error] Failed to resolve host name. Cancelling test suite.
[错误] 无法解析主机名。正在取消测试套件。
```

```
[error] Host resolve failed: Exec format error.
[错误] 主机解析失败: Exec 格式错误。
```

```
[error] Cannot open socket: No route to host.
[错误] 无法打开套接字: 没有到主机的路由。
```

```
[error] Server Selection - Failed to find a working test server. (NoServers)
[错误] 服务器选择 - 找不到可用的测试服务器。(NoServers)
```

## SSL 证书位置

默认情况下, 将在 Linux 机器上检查以下路径以查找 CA 证书束: 

```
/etc/ssl/certs/ca-certificates.crt
/etc/pki/tls/certs/ca-bundle.crt
/usr/share/ssl/certs/ca-bundle.crt
/usr/local/share/certs/ca-root-nss.crt
/etc/ssl/cert.pem
```

如果被测设备 **没有** 上述任何一个文件, 则可以手动将 curl 项目提供的规范且最新的 CA 证书束下载到特定位置。可以按照以下示例将此特定位置作为参数提供: 

```bash
wget https://curl.se/ca/cacert.pem
./ookla --ca-certificate=./cacert.pem
```
