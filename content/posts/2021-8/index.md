---
title: "Windows快速删除大量文件和文件夹"
date: 2021-03-19T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [CMD,PowerShell,Windows]
series: [Delete File and Directory]
series_weight: 2
featuredImagePreview: ""
summary: 使用 `del /f/s/q folder > nul`, `rd /s/q folder` 以及 `Remove-Item -LiteralPath 'folder' -Force -Recurse`。
---

当删除大量文件和文件夹时, 通过Windows资源管理器来删除, 速度是非常慢的。

{{< admonition type=quote title="Stack Overflow上的吐槽" open=true >}}
The worst way is to send to Recycle Bin: you still need to delete them. 

Next worst is shift+delete with Windows Explorer: it wastes loads of time checking the contents before starting deleting anything.
{{< /admonition >}}

## CMD

在cmd中, 最好的方法是使用以下命令: 

```batch
del /f/s/q folder > nul
rd /s/q folder
# 多个文件夹时
del /f/s/q folder1 folder2 > nul
rd /s/q folder1 folder2
# 文件夹名有空格时
del /f/s/q "folder name" > nul
rd /s/q "folder name"
```

### del

`del /f/s/q folder > nul`命令递归删除该目录下所有文件, 并将输出设为空, 以避免将每个文件的删除结果输出到屏幕的开销。

`del`命令的[官方文档](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771049(v=ws.11)), 有关上述参数的其他信息: 

- `/f`: 强制删除。
- `/s`: 递归删除。显示要删除的文件的名称。
- `/q`: 安静模式。删除时不提示用户确认删除。

### rd

{{< admonition type=info title="提示" open=true >}}
1. `rd`等同于`rmdir`。
2. 虽然`rd`或`rmdir`也可以直接递归删除目录和目录下所有文件, 看起来一步到位, 但是它的速度比`del`慢, 在文件非常多的情况下, 对比明显。
{{< /admonition >}}

`rd /s/q folder`命令清理剩余的目录结构 (这些目录已经空了, 也可以直接`Shift+Delete`删除)。

`rd`命令的[官方文档](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc726055(v=ws.11)), 有关上述参数的其他信息: 

- `/s`: 递归删除。
- `/q`: 安静模式。删除时不提示用户确认删除。 (请注意, 只有在指定了`/s`时, `/q`才有效。) 

## PowerShell

在PowerShell中, 最好的方法是使用以下命令: 

```shell
Remove-Item -Force -Recurse folder
# 多个文件夹时
Remove-Item -Force -Recurse folder1,folder2
# 文件夹名有空格时
Remove-Item -Force -Recurse "folder name"
```

推荐使用它们的别名, 简单易记: 

```shell
rm -Force -r folder
# 多个文件夹时
rm -Force -r folder1,folder2
# 文件夹名有空格时
rm -Force -r "folder name"
```

> 其实在PowerShell中, `rmdir`也是`Remove-Item`的别名。

`Remove-Item`命令的[官方文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item), 有关上述参数的其他信息: 

- LiteralPath: 指定一个或多个位置的路径。
- Force: 强制删除。
- Recurse: 递归删除。

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [What's the fastest way to delete a large folder in Windows?](https://stackoverflow.com/questions/186737/whats-the-fastest-way-to-delete-a-large-folder-in-windows/6208144)
2. [del 官方文档](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771049(v=ws.11))
3. [rd 官方文档](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc726055(v=ws.11))
4. [Remove-Item 官方文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item)
{{< /admonition >}}