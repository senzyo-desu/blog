---
title: "Linux清空文件内容的5种方法"
date: 2020-07-13T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Linux]
series: [Delete File and Directory]
series_weight: 1
featuredImagePreview: ""
summary: ①重定向到 `Null`; ②使用 `true` 重定向; ③使用 `cat` 或 `cp` 或`dd` 命令搭配 `/dev/null` 设备; ④使用 `echo`; ⑤使用 `truncate`。
---

> 在本文的示例中, 使用名为`access.log`的文件来作为示例样本。

## 重定向到Null

清空或者让一个文件成为空白的最简单方式, 是通过shell重定向`null` (不存在的事物) 到该文件: 

```shell
> access.log
```

## 使用true命令

`:`符号, 它是shell的一个内置命令, 等同于`true`命令, 它可被用来作为一个no-op (即不进行任何操作)。

下面将`:`或者`true`内置命令的输出重定向到文件中: 

```shell
: > access.log
```

或

```shell
true > access.log
```

## 使用cat或cp或dd命令搭配/dev/null设备

在Linux 中, `null`设备基本上被用来丢弃某个进程不再需要的输出流, 或者作为某个输入流的空白文件, 这些通常可以利用重定向机制来达到。所以`/dev/null`设备文件是一个特殊的文件, 它将清空送到它这里来的所有输入, 而它的输出则可被视为一个空文件。

- 所以, 可以使用`cat`命令显示`/dev/null`的内容然后重定向输出到某个文件, 以此来清空该文件: 

    ```shell
    cat /dev/null > access.log
    ```

- 使用`cp`命令复制`/dev/null`的内容到某个文件来清空该文件: 

    ```shell
    cp /dev/null access.log
    ```

- 使用`dd`命令, `if`代表输入文件, `of`代表输出文件: 

    ```shell
    dd if=/dev/null of=access.log
    ```

## 使用 echo 命令

使用`echo`命令将空字符串的内容重定向到文件中: 

```shell
echo "" > access.log
```

或

```shell
echo > access.log
```

{{< admonition type=note title="注意" open=true >}}
**空字符串**并不等同于`null`: 

- 字符串表明它是一个具体的事物, 只不过它的内容可能是空的。
- 但`null`则意味着某个事物并不存在。

基于这个原因, 将`echo`命令的输出作为输入重定向到文件后, 使用`cat`命令来查看该文件的内容时, 可以看到一个空白行 (即一个空字符串)。
{{< /admonition >}}

要将`null`做为输出输入到文件中, 可以使用`-n`选项, 这个选项将告诉`echo`不再像上面的那个命令那样输出结尾的那个新行: 

```shell
echo -n "" > access.log
```

## 使用 truncate 命令

`truncate`可以将一个文件缩小或者扩展到某个给定的大小。可以利用它和`-s`参数来特别指定文件的大小。要清空文件的内容, 则在下面的命令中将文件的大小设定为0: 

```shell
truncate -s 0 access.log
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
[5 Ways to Empty or Delete a Large File Content in Linux](http://www.tecmint.com/empty-delete-file-content-linux/)
{{< /admonition >}}
