---
title: "清除CMD和PowerShell的输入记录"
date: 2021-03-15T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [CMD,PowerShell,Windows]
featuredImagePreview: ""
summary: CMD 的输入历史记录不用处理。清除 PowerShell 的输入历史记录需要删除 `C:\Users\<UserName>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`。
---

## CMD

微软关于`doskey`命令的[官方文档](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/doskey)中写的很详细, 比如可以使用`doskey /history`查看当前CMD会话窗口的命令历史记录, 至于怎么清除历史记录? 

话说关闭当前CMD会话窗口, 再打开一个会话窗口, 就调不出来历史记录了。 

{{< admonition type=example title="仅供参考" open=true >}}
*不知道是不是我Windows版本的原因, 这些方法都没效果*: 

1. 改注册表
    在注册表中的`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU`下清除`默认`之外的所有字符串值。
    然而我电脑这里只有一个`默认`的字符串值。
2. 使用`doskey / listsize=0`
    让当前CMD会话窗口的命令历史记录条数为0, 间接实现`doskey /history`查不出历史记录的效果。
{{< /admonition >}}

## PowerShell

微软这个清除PowerShell的输入历史记录的[文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/clear-history), 根本没用啊。🙄

在Stack Overflow上也有这个问题: [PowerShell's Clear-History doesn't clear history](https://stackoverflow.com/questions/13257775/powershells-clear-history-doesnt-clear-history), 参考[高赞答案](https://stackoverflow.com/questions/13257775/powershells-clear-history-doesnt-clear-history/38807689#38807689)得知: 

{{< admonition type=quote title="原因" open=true >}}
PowerShell有自己的历史机制 (Get-History, Clear-History), 是独立于主机的, 所以还需要清除主机上存储的命令历史记录。
{{< /admonition >}}

所以, 

{{< admonition type=failure title="单独使用不行" open=true >}}
1. 只使用`Clear-History`命令

    虽然使用`Get-History`命令会发现只有一条历史记录, 就是刚刚输入的`Clear-History`。

    **但是**, 按几下`↑`键, 就会发现以前的输入记录都还在。
2. 只按`Alt + F7`

    虽然按几下`↑`键, 发现以前的输入记录都没有了。
    
    **但是**, 这时候使用`Get-History`命令会发现PowerShell把以前的历史记录清楚地列了出来。
{{< /admonition >}}

{{< admonition type=failure title="合起来使用也不行" open=true >}}
然而, 就算合起来使用这两种方法, 比如先按`Alt + F7`, 再使用`Clear-History`命令, 似乎这时候输入历史记录中就只有`Clear-History`这一条刚刚输入的命令了。

**但是**, 输入的命令历史记录跨会话保存在文件中, 即所谓的`PowerShell自己的历史机制是独立于主机的`。主机上存储的命令历史记录文件没有得到清除。
{{< /admonition >}}

### 解决方法

使用PowerShell执行以下命令: 

```shell
(Get-PSReadlineOption).HistorySavePath
```

会得到PowerShell的输入历史记录存放路径: `C:\Users\<UserName>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

打开一看, 历史记录全在这, 删除后, 关闭当前PowerShell, 再重新打开, 就调不出历史纪录了。
