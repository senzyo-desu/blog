---
title: "自动更新Algolia索引"
date: 2023-12-18T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [GitHub_Actions,Hugo]
featuredImagePreview: ""
summary: 通过 GitHub Actions 自动提交 JSON 来覆盖更新 Algolia 的索引, 不仅仅适用于 Hugo。
toc:
  enable: false
---

通过 GitHub Actions 自动提交 JSON 来覆盖更新 Algolia 的索引, 不仅仅适用于 Hugo。本文只是列举了使用 Hugo 的情况。

在 [Algolia Dashboard](https://dashboard.algolia.com/account/api-keys/) 的“Settings→API Keys”中复制 `Admin API Key`。

在项目仓库的“Settings→Secrets and variables→Actions”中新建一个 `Repository secrets`, 起名为 `ALGOLIA_ADMIN_KEY`, 粘贴刚刚复制的 `Admin API Key`。

在项目根目录下创建 `.github/workflows/algolia.yml` 文件, 写入以下内容:

```yaml
name: Update Algolia Index

on:
  push:
    branches:
      - master

jobs:
  update_algolia_index:
    name: Update Algolia Index
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --gc --minify

      - name: Update Algolia Index
        uses: tilburgsciencehub/algolia-uploader@v1
        with:
          app_id: <APP_ID>
          admin_key: ${{ secrets.ALGOLIA_ADMIN_KEY }}
          index_name: <INDEX_NAME>
          index_file_path: ./public/index.json
```

`app_id` 和 `index_name` 都改为自己的, 然后提交。

