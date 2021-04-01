---
title: "GitHub Repository Sync"
date: 2021-03-14T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Git,GitHub_Actions]
featuredImagePreview: ""
summary: 一个本地 Git 仓库使用多个 Remote 地址。使用 GitHub Actions 从上游同步已经 Fork 的仓库; 推送 GitHub 仓库到 GitLab。
---

## 多个Remote

让一个本地 Git Repository 使用多个 Remote, `git push` 时, 同时推送到多个 Remote, 参考 `git remote set-url --add <name> <newurl>`, 运行命令, 比如: 

```bash
git remote set-url --add origin git@gitlab.com:senzyo_sama/blog.git
```

使用 `git remote -v` 查看得到类似信息: 

```bash
origin  git@github.com:senzyo/blog.git (fetch)
origin  git@github.com:senzyo/blog.git (push)
origin  git@gitlab.com:senzyo_sama/blog.git (push)
```

这样在 `git push` 时, 就会同时推送到两个源。

## 从上游同步已经Fork的仓库

1. 打开 [https://github.com/settings/tokens](https://github.com/settings/tokens) 生成 personal access token (classic)。
2. 注意: Select scopes 勾选 `repo`, `workflow`。
3. 生成 token 并复制。
4. 在你专门用于运行 GitHub Actions 来同步的 Repository 中点击 **Settings** → **Secrets and variables** → **Actions** → **New repository secrets**。
5. `Name` 填写 `PERSONAL_TOKEN`, `Secret` 粘贴刚才的 token。
6. 按照你的需要编辑下面的 GitHub Actions 文件。

```yaml
name: Pull from upstream every 3 days

on:
  schedule:
    - cron: "0 20 */3 * *" # 4:00 AM UTC+8
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  ChatGemini:
    name: Sync ChatGemini
    runs-on: ubuntu-latest
    steps:
      - name: Sync ChatGemini
        run: |
          git clone --branch master https://github.com/bclswl0827/ChatGemini.git
          cd ChatGemini
          git remote set-url origin https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/ChatGemini.git
          git push -uf origin master:master

  github-readme-stats:
    name: Sync github-readme-stats
    runs-on: ubuntu-latest
    steps:
      - name: Sync github-readme-stats
        run: |
          git clone --branch master https://github.com/anuraghazra/github-readme-stats.git
          cd github-readme-stats
          git remote set-url origin https://${{ secrets.PERSONAL_TOKEN }}@github.com/senzyo/github-readme-stats.git
          git push -uf origin master:master
```

## 推送GitHub仓库到GitLab

1. 打开 [https://gitlab.com/-/user_settings/personal_access_tokens](https://gitlab.com/-/user_settings/personal_access_tokens) 生成个人访问令牌。
2. 注意: 选择范围勾选 `api`, `read_api`, `read_user`, `read_repository`, `write_repository`。
3. 生成 token 并复制。
4. 在你专门用于运行 GitHub Actions 来同步的 Repository 中点击 **Settings** → **Secrets and variables** → **Actions** → **New repository secrets**。
5. `Name` 填写 `GITLAB_PERSONAL_TOKEN`, `Secret` 粘贴刚才的 token。
6. 按照你的需要编辑下面的 GitHub Actions 文件。

```yaml
name: Push to GitLab every 3 days

on:
  schedule:
    - cron: "0 20 */3 * *" # 4:00 AM UTC+8
  workflow_dispatch:
  push:
    branches:
      - master

jobs:
  as-gist:
    name: Sync as-gist
    runs-on: ubuntu-latest
    steps:
      - name: Sync as-gist
        run: |
          git clone --branch master https://github.com/senzyo/as-gist.git
          cd as-gist
          git remote set-url origin https://oauth2:${{ secrets.GITLAB_PERSONAL_TOKEN }}@gitlab.com/senzyo_sama/as-gist.git
          git push -uf origin master:master

  blog:
    name: Sync blog
    runs-on: ubuntu-latest
    steps:
      - name: Sync blog
        run: |
          git clone --branch master https://github.com/senzyo/blog.git
          cd blog
          git remote set-url origin https://oauth2:${{ secrets.GITLAB_PERSONAL_TOKEN }}@gitlab.com/senzyo_sama/blog.git
          git push -uf origin master:master
```

