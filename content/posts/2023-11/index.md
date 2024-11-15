---
title: "自动更新Algolia索引"
date: 2023-12-18T09:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [GitHub]
featuredImagePreview: ""
summary: 通过 GitHub Actions 自动更新 Algolia 的索引。
toc:
  enable: false
---

以前使用 [Algolia Index Updater](https://github.com/marketplace/actions/algolia-index-updater), 但现在它已经无效了, 于是改用 Algolia 官方的 [Algolia Crawler Automatic Crawl](https://github.com/marketplace/actions/algolia-crawler-automatic-crawl)。

虽然 Algolia [官网](https://www.algolia.com/pricing/) 写着免费的 Build 套餐也有 `10k free crawls/month`, 但你如果在账号的 [Your plan and billing](https://dashboard.algolia.com/account/billing/overview?) 中发现没有 CRAWLER 的额度, 说明 Algolia 官方的设置出了岔子。

我的方法是重新注册账号, 选择 Build 套餐。

创建 Search 应用后, 进入左下角的 **Data sources** 依次点击 **Crawler** → **Domains** 绑定域名, 域名验证成功后可能创建一个 Crawler, 不用管, 可以删掉它。

在 Git 项目的根目录下创建 `.github/workflows/algolia.yml` 文件, 参考 [官方文档](https://github.com/marketplace/actions/algolia-crawler-automatic-crawl) 和以下内容: 

```yaml
name: Update Algolia Index

on:
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  algolia_recrawl:
    name: Algolia Recrawl
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        run: |
          version=$(curl -s https://api.github.com/repos/gohugoio/hugo/releases/latest | grep tag_name | cut -d "\"" -f4 | sed 's/v//')
          curl -fsSLo hugo.deb "https://github.com/gohugoio/hugo/releases/download/v${version}/hugo_extended_${version}_linux-amd64.deb"
          sudo dpkg -i hugo.deb

      - name: Build
        run: hugo --gc --minify

      - name: Algolia crawler creation and recrawl
        uses: algolia/algoliasearch-crawler-github-actions@v1
        id: crawler_push
        with:
          crawler-user-id: ${{ secrets.CRAWLER_USER_ID }}
          crawler-api-key: ${{ secrets.CRAWLER_API_KEY }}
          algolia-app-id: ${{ secrets.ALGOLIA_APP_ID }}
          algolia-api-key: ${{ secrets.ALGOLIA_API_KEY }}
          site-url: 'https://senzyo.net/posts/'
          override-config: true
```

再次来到 Algolia 的 **Data sources** → **Crawler** → **Settings** 页面, 

- 复制 `Crawler User Id` 作为 `CRAWLER_USER_ID`。
- 复制 `Crawler API Key` 作为 `CRAWLER_API_KEY`。

来到 [API Keys](https://dashboard.algolia.com/account/api-keys?) 页面, 

- 复制 `Application ID` 作为 `ALGOLIA_APP_ID`。
- 复制 `Write API Key` 作为 `ALGOLIA_API_KEY`。

在 GitHub 仓库的 **Settings** → **Secrets and variables** → **Actions** 中新建 `Repository secrets`, 填入以上几个 Secrets。

Push 项目到 GitHub 后, 如果 Action 成功运行, 应该可以看到 Algolia 的 **Data sources** → **Crawler** → **Crawlers** 页面新增了一个 Crawler。

{{< admonition type=warning title="注意" open=true >}}
**Search** → **Index** 页面会新增一个 `index`, 名称类似 `crawler_Github-senzyo-blog-refs-heads-master_index`, 这是 Crawler 自动创建的。

一开始创建的那个 `index` 已经用不上了, 所以记得在 Git 项目里改一下调用的 `index` 的名称。

然后再调整 **Search** → **Index** → **Configuration**。
{{< /admonition >}}
