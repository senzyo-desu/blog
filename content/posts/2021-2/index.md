---
title: "Hugo+Loveit优化记"
date: 2021-03-12T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Blog,Hugo]
featuredImagePreview: "/2021-2/featuredImagePreview.svg"
summary: 记录 LoveIt 配置中踩过的坑及处理办法。
---

> 此文中的LoveIt版本是[v0.2.10](https://github.com/dillonzq/LoveIt/releases/tag/v0.2.10)。

## 本地调试时加载评论系统

使用`hugo server`,会得到终端的提示: 

```shell
Current environment is "development". The "comment system", "CDN" and "fingerprint" will be disabled.
当前运行环境是 "development". "评论系统", "CDN" 和 "fingerprint" 不会启用.
```

{{< admonition type=success title="解决方法" open=true >}}
使用`hugo server -e production`命令即可运行生产环境进行调试, 就能加载评论系统了。
{{< /admonition >}}

## Console报错找不到`site.webmanifest`

{{< admonition type=warning title="一定要处理" open=true >}}
如果不处理的话, 会影响网站的打开速度。
{{< /admonition >}}

到[Favicon Generator](https://realfavicongenerator.net/)处理自己的网站图标, 最后会下载一个压缩包, 包括生成的图标和`browserconfig.xml`、`site.webmanifest`等文件, 将这些文件放到`blog\themes\LoveIt\static`中即可。

{{< admonition type=note title="顺便一提" open=true >}}
`blog\themes\LoveIt\static`这个目录里的文件, 最后会出现在网站根目录中。
{{< /admonition >}}

## LoveIt扩展Shortcodes

更多扩展Shortcodes的应用方法请查看LoveIt主题作者写的[使用说明](https://hugoloveit.com/zh-cn/theme-documentation-extended-shortcodes/)。

### admonition

admonition比较常用, 有12个样式, 但是主题作者并没说明每种样式对应的`type`是什么。我从源码中找到了它们的对应关系, 在此记录一下。

#### 用法

```markdown
{{</* admonition type=tip title="This is a tip" open=true */>}}
一个"技巧"横幅
{{</* /admonition */>}}
```

或者

```markdown
{{</* admonition tip "This is a tip" true */>}}
一个"技巧"横幅
{{</* /admonition */>}}
```

#### 示例

{{< admonition type=note title="注意" open=true >}}
type=note
{{< /admonition >}}

{{< admonition type=abstract title="摘要" open=true >}}
type=abstract
{{< /admonition >}}

{{< admonition type=info title="信息" open=true >}}
type=info
{{< /admonition >}}

{{< admonition type=tip title="技巧" open=true >}}
type=tip
{{< /admonition >}}

{{< admonition type=success title="成功" open=true >}}
type=success
{{< /admonition >}}

{{< admonition type=question title="问题" open=true >}}
type=question
{{< /admonition >}}

{{< admonition type=warning title="警告" open=true >}}
type=warning
{{< /admonition >}}

{{< admonition type=failure title="失败" open=true >}}
type=failure
{{< /admonition >}}

{{< admonition type=danger title="危险" open=true >}}
type=danger
{{< /admonition >}}

{{< admonition type=bug title="Bug" open=true >}}
type=Bug
{{< /admonition >}}

{{< admonition type=example title="示例" open=true >}}
type=example
{{< /admonition >}}

{{< admonition type=quote title="引用" open=true >}}
type=quote
{{< /admonition >}}

## keywords不生效

参考[Hugo系列(4) - 从Hexo迁移至Hugo以及使用LoveIt主题的踩坑记录](https://lewky.cn/posts/hugo-4.html/#网站配置了keywords没有生效)。

{{< admonition type=note title="前提配置" open=true >}}
在站点配置文件`config.toml`中填好网站关键词: 

```toml
  # 网站关键词
  keywords = "keyword1,keyword2"
```
{{< /admonition >}}

虽然已经设置了`keywords`, 但是F12查看网站源码后发现缺少`keywords`这个`meta`标签, 而且在[站长工具](https://seo.chinaz.com)里查询站点时发现页面TDK信息里的关键词`KeyWords`为空。

{{< admonition type=bug title="debug" open=true >}}
检查模板文件后发现是LoveIt主题没有引入该标签, 需要修改模板。
{{< /admonition >}}

### 解决方法

将`blog\themes\LoveIt\layouts\partials\head\meta.html`复制到`blog\layouts\partials\head\meta.html`, 打开该文件并在

```html
<meta name="Description" content="{{ $params.description | default .Site.Params.description }}">
```
的上方添加如下代码: 

```html
<meta name="keywords" content="{{ $params.keywords | default .Site.Params.keywords }}">
```

{{< admonition type=quote title="参考链接" open=true >}}
1. [Hugo系列(4) - 从Hexo迁移至Hugo以及使用LoveIt主题的踩坑记录](https://lewky.cn/posts/hugo-4.html/)
2. [Hugo系列(3.0) - LoveIt主题美化与博客功能增强 · 第一章](https://lewky.cn/posts/hugo-3.html/)
{{< /admonition >}}

{{< admonition type=info title="注意" open=true >}}
1. 上面两篇文章中的许多地方对于[LoveIt_v0.2.10](https://github.com/dillonzq/LoveIt/releases/tag/v0.2.10)是不必要的。
2. 大部分搜索引擎和网站都放弃使用meta keyword了, 参考[Pull request](https://github.com/HEIGE-PCloud/DoIt/pull/473), 
所以这keywords不设置也罢。
{{< /admonition >}}

## 换用twikoo评论系统

LoveIt主题没有适配适配[twikoo](https://twikoo.js.org/)评论系统, 所以需要手动设置一下。

{{< admonition type=tip title="建议使用DoIt主题" open=true >}}
🎉[DoIt](https://github.com/HEIGE-PCloud/DoIt)主题已适配twikoo🎉
{{< /admonition >}}

### 解决方法

可以修改评论系统模板文件`blog\themes\LoveIt\layouts\partials\comment.html`来手动添加对twikoo的支持, 在`<div id="comments"></div>`中添加以下代码: 

```html
{{- /* twikoo Comment System */ -}}
{{- $twikoo := $comment.twikoo}}
{{- if $twikoo.enable -}}
<div id="tcomment"></div>
<script src="https://cdn.jsdelivr.net/npmtwikoo@1.3.1/dist/twikoo.all.min.js"></script>
<script>
twikoo.init({
  envId: '',
  // 此处填写您的环境id
  el: '#tcomment',
  // region: 'ap-guangzhou', // 环境地域, 默认为 ap-shanghai, 如果您的环境地域不是上海, 需传此参数
  // path: 'window.location.pathname', // 用于区分不同文章的自定义 js 路径, 如果您的文章路径不是 location.pathname, 需传此参数
})
</script>
{{- end }}
```

然后在博客配置文件`blog\config.toml`中的

```toml
# 评论系统设置
[params.page.comment]
    enable = true
```

下面添加

```toml
# twikoo评论系统
[params.page.comment.twikoo]
    enable = true
```

{{< admonition type=tip title="更多twikoo配置" open=true >}}
云部署及版本更新等信息, 请到twikoo[官网](https://twikoo.js.org/)查看。
{{< /admonition >}}

### 自定义邮件通知模板

#### 博主通知邮件模板

适用场景: 
- 当博客上有新的评论时 (博主自己发表的评论除外), 系统发送邮件通知博主。
- 当博主发表的评论收到回复时, 系统发送邮件通知博主。

##### 效果图

<div style="padding:10px;margin:30px auto">
    <div
        style="border-radius:10px;font-size:15px;color:#000;font-family:'Century Gothic','Trebuchet MS','Hiragino Sans GB','微软雅黑','Microsoft Yahei',Tahoma,Helvetica,Arial,SimSun,sans-serif;border:1px solid #eee;box-shadow:0 1px 5px #cbc5b5">
        <div
            style="border-radius:10px 10px 0 0;background:linear-gradient(90deg,rgba(67,198,184,.5),rgba(255,209,244,.5))">
            <div style="border-radius:10px 10px 0 0;font-size:17px;color:#fff;word-break:break-all;padding:23px 32px">您在
                <a href="${POST_URL}" style="text-decoration:none;color:#12addb"
                    ;target="_blank"><strong>${SITE_NAME}</strong></a>&nbsp;发表的文章有新评论</div>
        </div>
        <div style="margin:40px auto;width:90%">
            <div><strong>${NICK}</strong>的评论如下: </div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${COMMENT}</div>
            <div>您可以查看<a style="text-decoration:none;color:#12addb" href="${POST_URL}" target="_blank">回复的完整內容</a>, 
                欢迎再次光临<a style="text-decoration:none;color:#12addb" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>。
            </div><br><br>
            <div style="font-size:13px;color:#909090">请注意: 此邮件由 <a style="text-decoration:none;color:#12addb"
                    href="${SITE_URL}" target="_blank">${SITE_NAME}</a> 自动发送, 请勿直接回复。</div>
        </div>
    </div>
</div>

##### 代码

```html
<div style="padding:10px;margin:30px auto">
    <div
        style="border-radius:10px;font-size:15px;color:#000;font-family:'Century Gothic','Trebuchet MS','Hiragino Sans GB','微软雅黑','Microsoft Yahei',Tahoma,Helvetica,Arial,SimSun,sans-serif;border:1px solid #eee;box-shadow:0 1px 5px #cbc5b5">
        <div
            style="border-radius:10px 10px 0 0;background:linear-gradient(90deg,rgba(67,198,184,.5),rgba(255,209,244,.5))">
            <div style="border-radius:10px 10px 0 0;font-size:17px;color:#fff;word-break:break-all;padding:23px 32px">您在
                <a href="${POST_URL}" style="text-decoration:none;color:#12addb"
                    ;target="_blank"><strong>${SITE_NAME}</strong></a>&nbsp;发表的文章有新评论</div>
        </div>
        <div style="margin:40px auto;width:90%">
            <div><strong>${NICK}</strong>的评论如下: </div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${COMMENT}</div>
            <div>您可以查看<a style="text-decoration:none;color:#12addb" href="${POST_URL}" target="_blank">回复的完整內容</a>, 
                欢迎再次光临<a style="text-decoration:none;color:#12addb" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>。
            </div><br><br>
            <div style="font-size:13px;color:#909090">请注意: 此邮件由 <a style="text-decoration:none;color:#12addb"
                    href="${SITE_URL}" target="_blank">${SITE_NAME}</a> 自动发送, 请勿直接回复。</div>
        </div>
    </div>
</div>
```

{{< admonition type=warning title="不要使用压缩后的代码" open=true >}}
如果将上面的代码压缩后, 再粘贴到`twikoo`管理面板中, 系统就不能发送邮件了, 不知道是哪里出了问题。

建议直接复制上面的代码粘贴使用。
{{< /admonition >}}

#### 普通通知邮件模板

适用场景: 
- 当访问者发表的评论收到回复时, 系统发送邮件通知被回复的访问者。

##### 效果图

<div style="padding:10px;margin:30px auto">
    <div
        style="border-radius:10px;font-size:15px;color:#000;font-family:'Century Gothic','Trebuchet MS','Hiragino Sans GB','微软雅黑','Microsoft Yahei',Tahoma,Helvetica,Arial,SimSun,sans-serif;border:1px solid #eee;box-shadow:0 1px 5px #cbc5b5">
        <div
            style="border-radius:10px 10px 0 0;background:linear-gradient(90deg,rgba(67,198,184,.5),rgba(255,209,244,.5))">
            <div style="border-radius:10px 10px 0 0;font-size:17px;color:#fff;word-break:break-all;padding:23px 32px">
                您在
                <a href="${POST_URL}" style="text-decoration:none;color:#12addb"
                    ;target="_blank"><strong>${SITE_NAME}</strong></a>&nbsp;的评论收到新的回复
            </div>
        </div>
        <div style="margin:40px auto;width:90%">
            <div><strong>${PARENT_NICK}</strong>, 您曾发表评论:</div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${PARENT_COMMENT}</div>
            <div><strong>${NICK}</strong>的回复如下: </div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${COMMENT}</div>
            <div>您可以查看<a style="text-decoration:none;color:#12addb" href="${POST_URL}" target="_blank">回复的完整內容</a>, 
                欢迎再次光临<a style="text-decoration:none;color:#12addb" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>。
            </div><br><br>
            <div style="font-size:13px;color:#909090">请注意: 此邮件由 <a style="text-decoration:none;color:#12addb"
                    href="${SITE_URL}" target="_blank">${SITE_NAME}</a> 自动发送, 请勿直接回复。</div>
        </div>
    </div>
</div>

##### 代码

```html
<div style="padding:10px;margin:30px auto">
    <div
        style="border-radius:10px;font-size:15px;color:#000;font-family:'Century Gothic','Trebuchet MS','Hiragino Sans GB','微软雅黑','Microsoft Yahei',Tahoma,Helvetica,Arial,SimSun,sans-serif;border:1px solid #eee;box-shadow:0 1px 5px #cbc5b5">
        <div
            style="border-radius:10px 10px 0 0;background:linear-gradient(90deg,rgba(67,198,184,.5),rgba(255,209,244,.5))">
            <div style="border-radius:10px 10px 0 0;font-size:17px;color:#fff;word-break:break-all;padding:23px 32px">
                您在
                <a href="${POST_URL}" style="text-decoration:none;color:#12addb"
                    ;target="_blank"><strong>${SITE_NAME}</strong></a>&nbsp;的评论收到新的回复
            </div>
        </div>
        <div style="margin:40px auto;width:90%">
            <div><strong>${PARENT_NICK}</strong>, 您曾发表评论:</div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${PARENT_COMMENT}</div>
            <div><strong>${NICK}</strong>的回复如下: </div>
            <div
                style="border-radius:5px;background-color:#fafafa;background-image:repeating-linear-gradient(-45deg,#fff,#fff 15px,transparent 15px,transparent 30px);box-shadow:0 2px 5px #cbc5b5;margin:20px 0;padding:15px">
                ${COMMENT}</div>
            <div>您可以查看<a style="text-decoration:none;color:#12addb" href="${POST_URL}" target="_blank">回复的完整內容</a>, 
                欢迎再次光临<a style="text-decoration:none;color:#12addb" href="${SITE_URL}" target="_blank">${SITE_NAME}</a>。
            </div><br><br>
            <div style="font-size:13px;color:#909090">请注意: 此邮件由 <a style="text-decoration:none;color:#12addb"
                    href="${SITE_URL}" target="_blank">${SITE_NAME}</a> 自动发送, 请勿直接回复。</div>
        </div>
    </div>
</div>
```

{{< admonition type=warning title="再次提示, 不要使用压缩后的代码" open=true >}}
如果将上面的代码压缩后, 再粘贴到`twikoo`管理面板中, 系统就不能发送邮件了, 不知道是哪里出了问题。

建议直接复制上面的代码粘贴使用。
{{< /admonition >}}