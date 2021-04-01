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

### Debian

1. 安装必要组件: 

   ```bash
   sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release
   ```

2. 添加GPG: 

   ```bash
   curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
   ```

3. 设置Node.js版本: 

   ```bash
   NODE_MAJOR=20
   ```

4. 设置Node.js源: 

   ```bash
   echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
   ```

5. 安装Node.js: 

   ```bash
   sudo apt-get update && sudo apt-get install nodejs -y
   ```

6. 检查版本: 

   ```shell
   node --version
   npm --version
   ```

### Windows

最新尝鲜版: [https://nodejs.org/en/download/current](https://nodejs.org/en/download/current)

长期维护版: [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

所有版本: [https://nodejs.org/dist/](https://nodejs.org/dist/)

习惯使用`zip`而不是`msi`方式安装Node.js。

## NPM

### Install

npm包含在Node.js的msi或者zip中, 不必单独安装。

### Config

#### Environment Variable

对于Windows:

1. Node.js文件夹路径: `D:\NodeJS`。
2. 创建文件夹`D:\NodeJS\node_cache`和`D:\NodeJS\node_global`。
3. **环境变量**→**系统变量**→**PATH**中, 新建`D:\NodeJS`, `D:\NodeJS\node_cache`和`D:\NodeJS\node_global`。

#### Cache & Global

检查npm的缓存路径和全局安装路径: 

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

2. 添加GPG: 

   ```bash
   curl -fsSL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/yarn.gpg
   ```

3. 设置Yarn源: 

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/yarn.gpg] https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
   ```

4. 安装Yarn: 

   ```bash
   sudo apt-get update && sudo apt-get install -y yarn
   ```

5. 检查版本: 

   ```bash
   yarn --version
   ```

### Config

#### Environment Variable

对于Windows:

1. 创建文件夹`D:\NodeJS\yarn_cache`和`D:\NodeJS\yarn_global`。
2. **环境变量**→**系统变量**→**PATH**中, 新建`D:\NodeJS\yarn_cache`和`D:\NodeJS\yarn_global`。

#### Cache & Global

检查yarn的缓存路径和全局安装路径: 

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
