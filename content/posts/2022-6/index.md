---
title: "Cloudflare使用问题"
date: 2022-03-05T11:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Cloudflare,Help,NodeJS]
featuredImagePreview: "/2022-6/featuredImagePreview.svg"
summary: 记录使用 Cloudflare 时遇到的问题。
---

## JS文件已加载但不生效

本博客以前采用的评论系统是 [Twikoo](https://github.com/imaegoo/twikoo), 有次出现了奇怪的问题: 

通过浏览器 Web 开发者工具可以看到已经加载了 `twikoo.all.min.js` 文件, 云函数的数据库集合 `counter` 中也显示刚才浏览的文章的阅读数已增加, 但是文章下方并不显示评论框, 只有刷新网页后才显示。

我排查过很多方面, 甚至一度以为已经解决了问题, 但后来问题再次出现。最终发现问题是 Cloudflare 的 JS 异步加载设置导致的。

解决方法: Cloudflare 网站 → 速度 → 优化 → 内容优化 → 禁用 `Rocket Loader`。

## Cloudflare Pages使用Yarn出错

在 Cloudflare Pages 中使用 Yarn 来构建项目时总是失败, 明明在本地和 GitHub Actions 中都能成功构建。

参考 Cloudflare 论坛的这则 [帖子](https://community.cloudflare.com/t/react-app-cant-build-in-pages/565639), 原因是用户使用的 Yarn 版本和 Cloudflare 使用的不同。

解决方法是为 Cloudflare Pages 项目设置环境变量 `YARN_VERSION` = `1`。