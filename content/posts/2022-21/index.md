---
title: "Apktool安装与简单使用"
date: 2022-12-02T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Android]
featuredImagePreview: ""
summary: 这是一篇许久以前的 Apktool 使用笔记。
---

## Install

https://ibotpeaches.github.io/Apktool/install/

### Environment

需要有 java 1.8 或者更高环境。

### Windows

1. 下载脚本 [wrapper script](https://ghproxy.net/https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/windows/apktool.bat) 重命名为 `apktool.bat`
2. 下载 jar 包 [find newest here](https://bitbucket.org/iBotPeaches/apktool/downloads/)
3. 重命名 jar 包为 `apktool.jar`
4. 将 `apktool.jar` 和 `apktool.bat` 移动至某一文件夹下, 比如 `D:\Apktool`
5. 将 `D:\Apktool` 添加到系统变量的 `PATH` 中
6. 通过命令行使用 `apktool`, 比如 `apktool --version`

### Linux

1. 下载脚本 [wrapper script](https://ghproxy.net/https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool) 重命名为 `apktool`
2. 下载 jar 包 [find newest here](https://bitbucket.org/iBotPeaches/apktool/downloads/)
3. 重命名 jar 包为 `apktool.jar`
4. 将 `apktool.jar` 和 `apktool` 移动至某一文件夹下, 比如 `/usr/local/bin` (此路径需要 `root` 用户)
5. 确保 `apktool.jar` 和 `apktool` 都具有可执行权限( `chmod +x` )
6. 通过命令行使用 `apktool`, 比如 `apktool --version`

## Decode

[Apktool - Documentation](https://ibotpeaches.github.io/Apktool/documentation/)

```shell
apktool d bar.apk
# decodes bar.apk to bar folder

apktool d bar.apk -o baz
# decodes bar.apk to baz folder
```

## Build

```shell
apktool b bar
# builds bar folder into bar/dist/bar.apk file

apktool b .
# builds current directory into ./dist

apktool b bar -o new_bar.apk
# builds bar folder into new_bar.apk
```

## Signature

使用 `jarsigner` 或者 `apksigner` 对 apk 进行签名。反正是这个 apk 自己用, 怎么方便怎么来, 支不支持 V2 无所谓。

### Use jarsigner

`jarsigner` 只支持 V1 签名。

1. 使用此条命令, 会生成 `keystore`  文件, 我将其命名为 `senzyo.keystore`, 别名 `alias` 命名为 `senzyo`

```shell
keytool -genkey -keystore senzyo.keystore -alias senzyo -keyalg RSA -keysize 4096 -validity 10000 
```

2. 使用刚才生成的 `keystore`  文件进行签名, `PureIcon_signed.apk` 是签名后的apk名称, `PureIcon.apk` 是将要签名(还未签名)的apk。

```shell
jarsigner -sigalg MD5withRSA -digestalg SHA1 -keystore senzyo.keystore -signedjar PureIcon_signed.apk PureIcon.apk senzyo -tsa http://sha256timestamp.ws.symantec.com/sha256/timestamp
```

`-tsa` 后的网址是用来增加时间戳的

### Use apksigner

`apksigner` 默认同时使用 V1 和 V2 签名。

参考 [apksigner](https://developer.android.com/studio/command-line/apksigner?hl=zh-cn) 与 [jarsigner 和 apksigner 详解](https://cloud.tencent.com/developer/article/1743269)

### Use MT 管理器

支持 V1, V2 和 V3 签名, 操作简单。

## Optimize

[Android 开发者网站](https://developer.android.google.cn/studio/command-line/zipalign.html) 对于 `zipalign` 的介绍: 

> zipalign 是一种 zip 归档文件对齐工具。它可确保归档中的所有未压缩文件相对于文件开头都是对齐的。在将 APK 文件分发给最终用户之前, 应该先使用 zipalign 进行优化。如果使用 Android Studio 进行构建, 则此步骤会自动完成。

`PureIcon_signed.apk` 是签名后的apk, `PureIcon_zipaligned.apk` 是经过 zipalign 优化后的apk。

```shell
zipalign -v 4 PureIcon_signed.apk PureIcon_zipaligned.apk
```

**注意**

- 若使用 `jarsigner` 对 apk 进行签名, 需要签名**后**使用 `zipalign`。
- 若使用 `apksigner` 对 apk 进行签名, 需要在签名**前**使用 `zipalign`。

## Done

现在可以安装到手机上了。