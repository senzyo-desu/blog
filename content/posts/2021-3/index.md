---
title: "GitHub Actions部署Hugo站点"
date: 2021-03-13T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [GitHub,Hugo]
featuredImagePreview: "/2021-3/featuredImagePreview.svg"
summary: GitHub Actions 双分支或双仓库模式部署 Hugo 站点。
---

## GitHub Actions双分支模式

1. 创建一个公有仓库, 用来存放博客源码。
2. Hugo 生成的 `public` 文件夹放到另一个分支作为 GitHub Pages 发布。
3. 在博客源码仓库的根目录下创建 `.github/workflows/gh-pages.yml` 文件, 使用以下代码, 然后提交: 

```yaml
name: GitHub Pages

on:
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  Deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true
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

      - name: Public
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: gh-pages
          publish_dir: ./public
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          full_commit_message: ${{ github.event.head_commit.message }}
          # 填写自定义域名
          # cname: 
```

如果 CNAME 使用 `www` 域名前缀, 参考 [GitHub Pages使用问题](../2022-5/)。

## GitHub Actions双仓库模式

1. 创建一个私有仓库, 用来存放博客源码。
2. 创建一个公有仓库, 用来发布 Hugo 生成的 `public` 文件夹。
3. 创建 [Personal access tokens](https://github.com/settings/tokens), 点击 `Generate new token (classic) `。
4. Note 随便填。
5. Expiration 选择 `No expiration`。
6. Select scopes 勾选 `repo` 和 `workflow`。
7. Generate token, 复制它的值。
8. 到博客源码仓库的“Settings→Secrets and variables→Actions”中新建一个 `Repository secrets`。
9. Name 设置为 `PERSONAL_TOKEN`, Value 粘贴上面 Personal access tokens 的值。
10. 在博客源码仓库的根目录下创建 `.github/workflows/gh-pages.yml` 文件, 使用以下代码, 然后提交: 

```yaml
name: GitHub Pages

on:
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  Deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
      cancel-in-progress: true
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

      - name: Public
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          # 填写用来发布博客的公有仓库名称
          external_repository: <UserName/external-repository>
          publish_branch: gh-pages
          publish_dir: ./public
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          full_commit_message: ${{ github.event.head_commit.message }}
          # 填写自定义域名
          # cname: 
```

如果 CNAME 使用 `www` 域名前缀, 参考 [GitHub Pages使用问题](../2022-5/)。

## 其他部署方式

1. [CircleCI](https://circleci.com/)、[Cloudflare Pages](https://pages.cloudflare.com/)、[Netlify](https://www.netlify.com/)、[Travis CI](https://www.travis-ci.com/)、[Vercel](https://vercel.com/) 等。

{{< admonition type=warning title="警告" open=true >}}
Cloudflare Pages 的 `.pages.dev` 已被墙, 如果使用 Cloudflare Pages, 建议自定义域名, 然后 DNS 设置为通过 Cloudflare 代理。
{{< /admonition >}}

2. 使用 Nginx 在自己服务器上部署, 参考 [Nginx部署Hugo站点](../2022-8/)。

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo)
2. [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
{{< /admonition >}}
