---
title: "GitHub Pages使用问题"
date: 2022-03-04T20:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [GitHub,Help]
featuredImagePreview: ""
summary: 记录在使用 GitHub Pages 时遇到的问题, 比如自定义域名设置失败。
---

## 自定义域名已被占用

当你为某个GitHub存储库的Pages设置了自定义域名, 之后出于某些原因直接删掉了这个存储库时, 那么你**有可能**遇到一个糟糕的错误——你无法为其他GitHub存储库的Pages配置这个自定义域名, 并且得到一个提示: 

> The custom domain www.example.com is already taken. If you are the owner of this domain, check out https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages for information about how to verify and release this domain.



这种情况在GitHub Community里也有人提出过: 

- https://github.community/t/the-cname-is-already-taken/149785
- https://github.community/t/the-custom-domain-is-already-taken-except-its-not/215984

### 解决方法

1. 在[https://github.com/settings/pages](https://github.com/settings/pages)里验证这个域名的所有权。
2. 在[GitHub Support](https://support.github.com/request/pages)选择**Help with a custom domain**。
3. 描述自己的问题。
4. (对应步骤1) 说明自己已经验证该域名的所有权, 不然还会被要求验证一遍。
5. 说明那个设置了自定义域名的、已经被你删掉的存储库的名称。
6. 如果那个自定义域名不只被一个存储库使用, 那就把它们的名称都说明一下。

如无意外, 之后GitHub Support的工作人员会表示已解决问题: 

{{< admonition type=quote title="回复" open=true >}}
I've purged the data from your deleted repos.

Could you try and add your custom domain again? Let me know how it goes!

I'll wait to hear back from you.
{{< /admonition >}}

{{< admonition type=tip title="避免问题再次出现" open=true >}}
稳妥的操作是: 先移除Pages的自定义域名, 之后删除存储库。
{{< /admonition >}}

## www域名设置失败

明明其他子域名如`blog`等都可以正确的设置, 但是`www`域名就是不能设置成功: 

{{< admonition type=failure title="失败提示" open=true >}}
example.com is improperly configured

Domain does not resolve to the GitHub Pages server. For more information, see Learn more (NotServedByPagesError). 

We recommend you add an A record pointed to our IP addresses, or an ALIAS record pointing to username.github.io.
{{< /admonition >}}

按照这个提示添加了DNS的A记录后, 似乎也没有用, 甚至可能会得到另外一个报错; 另一方面, 有些域名托管商也不支持ALIAS记录。

### 解决方法

网络上搜索到的大部分回答都是“等待DNS传播生效, 最多48小时”, 其实DNS传播生效不用那么长时间, 除非自己把DNS的TTL设置的特别长。另一方面, 对于这个问题, 只靠等待是解决不了了的, 参考[Custom domains in GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages), 应该像下面这样设置域名: 

| 类型  | 名称        | 内容               |
| ----- | ----------- | ------------------ |
| CNAME | www         | username.github.io |
| CNAME | example.com | www.example.com    |

