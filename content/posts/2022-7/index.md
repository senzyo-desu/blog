---
title: "浏览IE网页的兼容性问题"
date: 2022-03-07T11:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Browser,Help,Windows]
featuredImagePreview: "/2022-7/featuredImagePreview.svg"
summary: 在 Windows10 和 Windows11 中, 使用 Internet Explorer、Edge 和 Chrome 浏览只支持 Internet Explorer 的网页。
---

> 本文以常见的**中小学教师资格考试**报名系统作为示例, 省份选择山东省。

## 常见问题

通过[NTCE - 中国教育考试网](http://ntce.neea.edu.cn/)进入**报名系统**: 

{{< image src="1.png" caption="中小学教师资格考试" height="auto" width="80%">}}

选择省份, 点击登录: 

{{< image src="2.png" caption="选择某一省份" height="auto" width="80%">}}

问题来了: 

{{< image src="3.png" caption="经典的界面" height="auto" width="80%">}}

## Windows10

对于Windows10, 可以直接使用Windows10自带的Internet Explorer 11, 只不过浏览器默认内核不一定是哪个版本, 如果是IE11, 那么还是打不开这个网站。

### Internet Explorer 11

Windows10默认是有Internet Explorer 11的, 如果没有, 可以搜索**启用或关闭Windows功能**: 

{{< image src="4.png" caption="启用或关闭Windows功能" height="auto" width="50%">}}

启用Internet Explorer 11: 

{{< image src="5.png" caption="启用Internet Explorer 11" height="auto" width="50%">}}

#### 方法一

按`ALT+X+B`, 调出**兼容性视图设置**, 添加当前网站: 

{{< image src="6.png" caption="兼容性视图设置" height="auto" width="50%">}}

这样Internet Explorer 11会使用低版本内核来打开该网站。

#### 方法二

按`F12`调出开发者工具, 在**仿真**一栏, 更改**文档模式**, 选择其他内核版本: 

{{< image src="7.png" caption="Internet Explorer开发者工具" height="auto" width="80%">}}

最好点一下左上角**保留仿真设置**, 不然浏览器打开其他标签页或其他网址, 还是会调用默认内核。这样Internet Explorer 11会使用被选中的内核来打开该网站。

## Windows11

> 以下方法也适用于Windows10, 只不过在Windows10直接用Internet Explorer更方便。

### Edge

在[设置→默认浏览器](edge://settings/defaultBrowser)中, 将相应网址添加进**Internet Explorer 模式页面**: 

{{< image src="8.png" caption="Internet Explorer 模式页面" height="auto" width="80%">}}

一般来说可以打开网站了。

然而, [http://ntcebm7.neea.edu.cn/apply/memapp/ieNote](http://ntcebm7.neea.edu.cn/apply/memapp/ieNote)这个网址只是专门用于提示, 就算使用Internet Explorer模式浏览此页面, 它也不会跳转到正确的入口。

{{< image src="9.png" caption="此页面只用于提示" height="auto" width="80%">}}

另外, 由于**Internet Explorer 模式页面**仅对网址生效, 不支持域名或者通配符方式, 所以这时只有添加正确的入口网址才可以。但是, 本来就进不去正确的入口, 怎么得到它的网址? 死循环吗? 并不, 我们可以直接将**点击登录**按钮的跳转网址[http://ntcebm7.neea.edu.cn/apply/](http://ntcebm7.neea.edu.cn/apply/)添加进**Internet Explorer 模式页面**列表: 

{{< image src="10.png" caption="选择省份跳转登录页面" height="auto" width="80%">}}

{{< image src="11.png" caption="Internet Explorer 模式页面" height="auto" width="80%">}}

再次点击**点击登录**按钮, 就会跳转到正确的入口: [http://ntcebm7.neea.edu.cn/apply/memapp/memLogin](http://ntcebm7.neea.edu.cn/apply/memapp/memLogin)

{{< admonition type=failure title="还是不行? " open=true >}}
因为有些人的Edge默认使用IE11内核, 而这个报名系统又不支持IE11, 所以要切换内核, IE11以下版本皆可。
{{< /admonition >}}

#### 切换内核

打开IEChooser: 

1. 按`Windows` + `R`打开**运行**对话框。
2. 输入`%systemroot%\system32\f12\IEChooser.exe`, 然后确定。

选择`ieNote`页面, 即http://ntcebm7.neea.edu.cn/apply/memapp/ieNote: 

{{< image src="12.png" caption="IEChooser" height="auto" width="80%">}}

点击**仿真**一栏: 

{{< image src="13.png" caption="IEChooser" height="auto" width="80%">}}

点击左上角**保留仿真设置**, 更改**文档模式** (即更改IE内核版本), IE11以下版本皆可。

{{< admonition type=note title="注意" open=true >}}
如果不保留仿真设置, 关闭IEChooser调试页面后, 网页会刷新, 并使用默认IE内核, 可能是IE11, 还是无法进入正确的入口页面。

当然可以在使用此报名系统时, 保持IEChooser调试页面处于打开状态, 以防内核设置被覆盖。
{{< /admonition >}}

现在通过[http://ntcebm7.neea.edu.cn/apply/](http://ntcebm7.neea.edu.cn/apply/)即可进入正确的入口: 

{{< image src="14.png" caption="正确的登录入口" height="auto" width="80%">}}

{{< admonition type=bug title="Edge抽风" open=false >}}
不知什么原因, Edge调用IE内核版本时可能有点抽风, 如果你设置过IEChooser调试页面, 即使保留了仿真设置, 当再次打开Edge时, 它**有可能**会默认调用IE11或者IE7, 甚至是IE5。

{{< image src="15.png" caption="抽风的Edge和它的IE版本" height="auto" width="100%">}}
{{< /admonition >}}

### Chrome

使用扩展: [IE Tab](https://chrome.google.com/webstore/detail/ie-tab/hehijbfgiekmjfkfjpbkbammjbdenadd), 或者[IEability](https://chrome.google.com/webstore/detail/ieability-open-in-ie/moffahdcgnjnglbepimcggkjacdmpojc/)。

如果打不开网页, 试一下切换IE内核。
