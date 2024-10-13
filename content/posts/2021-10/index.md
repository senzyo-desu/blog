---
title: "NPM与Yarn的安装与配置"
date: 2021-03-24T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Custom,NodeJS]
featuredImagePreview: ""
summary: Debian 系 Linux 和 Windows 中, NodeJS, NPM 与 Yarn 的安装与配置。
---

## Install NodeJS

### Linux

#### Use nvm

https://github.com/nvm-sh/nvm?tab=readme-ov-file#git-install

如果无法稳定快速地访问 GitHub, 使用文件 `https://raw.hellogithub.com/hosts` 修改系统 hosts: 

- Windows: `C:\Windows\System32\drivers\etc\hosts`
- Linux: `/etc/hosts`

#### Build from Source Code

https://github.com/nodesource/distributions

### Windows

预构建安装程序: https://nodejs.org/zh-cn/download/prebuilt-installer/current

预构建二进制文件 (压缩包): https://nodejs.org/zh-cn/download/prebuilt-binaries/current

所有版本: https://nodejs.org/dist/

## NPM

### Install

npm 已被包含在 Node.js 中, 不必单独安装。

### Config

#### Environment Variable

对于Windows:

1. Node.js文件夹路径: `D:\NodeJS`。
2. 创建文件夹 `D:\NodeJS\node_cache` 和 `D:\NodeJS\node_global`。
3. **环境变量** → **系统变量** → **PATH** 中, 新建 `D:\NodeJS`, `D:\NodeJS\node_cache` 和 `D:\NodeJS\node_global`。

#### Cache & Global

检查 npm 的缓存路径和全局安装路径: 

```shell
npm config get cache
npm config get prefix
```

配置缓存路径和全局安装路径: 

```shell
npm config set cache "D:\NodeJS\node_cache"
npm config set prefix "D:\NodeJS\node_global"
```

#### NPM Mirrors

检查当前源: 

```shell
npm config get registry
```

更换镜像源: 

```shell
# 腾讯云镜像
npm config set registry https://mirrors.cloud.tencent.com/npm/
# 淘宝镜像源
npm config set registry https://registry.npmmirror.com
# 默认官方源
npm config set registry https://registry.npmjs.org
```

## Yarn

### Install

#### Via NPM

```shell
npm install --global yarn
```

检查版本:

```shell
yarn --version
```

#### Via Official Package Repository

1. 安装必要组件: 

   ```bash
   sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
   ```

2. 添加 GPG: 

   ```bash
   curl -fsSL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/yarn.gpg
   ```

3. 设置 Yarn 源: 

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
   ```

4. 安装 Yarn: 

   ```bash
   sudo apt-get update && sudo apt-get install -y yarn
   ```

5. 检查版本: 

   ```bash
   yarn --version
   ```

### Config

#### Environment Variable

对于 Windows:

1. 创建文件夹 `D:\NodeJS\yarn_cache` 和 `D:\NodeJS\yarn_global`。
2. **环境变量** → **系统变量** → **PATH** 中, 新建 `D:\NodeJS\yarn_cache` 和 `D:\NodeJS\yarn_global`。

#### Cache & Global

检查 yarn 的缓存路径和全局安装路径: 

```shell
yarn config get cache-folder
yarn config get global-dir
```

配置缓存路径和全局安装路径: 

```shell
yarn config set cache-folder "D:\NodeJS\yarn_cache"
yarn config set global-dir "D:\NodeJS\yarn_global"
```

#### Yarn Mirrors

检查当前源: 

```shell
yarn config get registry
```

更换镜像源: 

```shell
# 腾讯云镜像
yarn config set registry https://mirrors.cloud.tencent.com/npm/
# 淘宝镜像源
yarn config set registry https://registry.npmmirror.com
# 默认官方源
yarn config set registry https://registry.yarnpkg.com
```
