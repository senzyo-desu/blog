---
title: "自定义CMD提示符并着色"
date: 2022-12-17T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [CMD,Custom,Windows]
series: [Customize Prompt of Shell]
series_weight: 3
featuredImagePreview: ""
summary: Customize and Colorize CMD Prompt.
---

## ANSI转义字符

和 Bash 以及 PowerShell 类似, 要控制显示格式, 须使用 SGR 转义序列: `ESC[<n>m`。简单理解, 以 `ESC[` 开头; 中间 `<n>` 处设置颜色, 使用多个代码时中间用分号`;`隔开, 比如`1;31`; 以 `m` 结尾。

## 显示控制代码

显示控制代码有3类:

- 效果控制代码
- 前景色控制代码 (即字体颜色) 
- 背景色控制代码

### 效果控制代码

| 代码  | 效果                       |
| :---: | -------------------------- |
|   0   | 重置所有显示属性为默认设置 |
|   1   | 字体加粗                   |
|   2   | 字体正常粗细               |
|   4   | 字体加下划线               |
|   5   | 字体闪烁                   |
|   7   | 前景色与背景色调转         |
|   8   | 隐藏                       |

### 前景色与背景色控制代码

| 前景色代码 | 背景色代码 | 颜色  |
| :--------: | :--------: | :---: |
|     30     |     40     | 黑色  |
|     31     |     41     | 红色  |
|     32     |     42     | 绿色  |
|     33     |     43     | 黄色  |
|     34     |     44     | 蓝色  |
|     35     |     45     | 品红  |
|     36     |     46     | 青色  |
|     37     |     47     | 白色  |

## Prompt代码

参考 [这里](../2020-1/#永久代理) 设置永久配置 `cmdrc.cmd`, 将下面的代码写入, 使其在打开 cmd 时可以被预加载。

```batch
@echo off
:: Prompt
prompt $_$e[1;37m$T $e[1;31m用户名$e[1;37m@$e[1;31m电脑名 $e[1;36m$P $_$e[1;31m$$ $e[0m
```

### 解释

在 cmd 中运行命令 `help prompt` 得知, 提示符可以由普通字符及下列特殊代码组成:

| 字符  |            描述            |
| :---: | :------------------------: |
|  $A   |          & (与号)          |
|  $B   |         \| (竖线)          |
|  $C   |         ( (左括号)         |
|  $D   |          当前日期          |
|  $E   |    转义码(ASCII 码 27)     |
|  $F   |         ) (右括号)         |
|  $G   |         > (大于号)         |
|  $H   | Backspace (删除前一个字符) |
|  $L   |         < (小于号)         |
|  $N   |         当前驱动器         |
|  $P   |      当前驱动器及路径      |
|  $Q   |          = (等号)          |
|  $S   |           (空格)           |
|  $T   |          当前时间          |
|  $V   |       Windows 版本号       |
|  $_   |         回车换行符         |
|  $$   |        $ (美元符号)        |

如果命令扩展被启用, PROMPT 命令会支持下列格式化字符:

| 字符  |                                    描述                                    |
| :---: | :------------------------------------------------------------------------: |
|  $+   | 根据 PUSHD 目录堆栈的深度, 零个或零个以上加号(+)字符, 一个推的层一个字符。 |
|  $M   | 如果当前驱动器不是网络驱动器, 显示跟当前驱动器号或空字符串有关联的远程名。 |

### 效果

<div style="background-color:black">
    <br /><span style="color:white;">20:34:50.71</span> <span style="color:red;">senzyo</span><span style="color:white;">@</span><span style="color:red;">Windows</span> <span style="color:cyan;">D:\Desktop</span><div></div><span style="color:red;">$ </span>
</div>

- `$T` 会将毫秒一起输出, 暂时没有找到控制其输出格式的命令
- 遗憾的是不能像 Bash 和 PowerShell 那样根据当前用户身份分别显示用户名和 `@`, `#`

--------------------

{{< admonition type=tip title="顺便一提" open=true >}}
PowerShell 后跟启动参数 `-nologo`, CMD 后跟启动参数 `/k`, 可以取消显示顶端的版本和版权信息。
{{< /admonition >}}