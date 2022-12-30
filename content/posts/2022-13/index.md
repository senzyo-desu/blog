---
title: "Nginx防盗链"
date: 2022-04-08T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Nginx,Security]
series: [Security Config of Nginx]
series_weight: 4
featuredImagePreview: ""
summary: 使用 Nginx 自带模块 `ngx_http_referer_module`, 基于 `http_referer` 判定非法请求, 从而阻止连接。能防盗链, 但只能防一点点, 毕竟模拟 `http_referer` 不是难事。
toc:
  enable: false
---

```nginx
location ~ .*\.(bmp|gif|ico|jpeg|jpg|png|webp|svg)$ {
    ...
    valid_referers none blocked server_names 
        *.example.com www.bing.com ~\.google\.;
    if ($invalid_referer) {
        return 444;
    }
    ...
}
```

**$invalid_referer可跟参数**: 

- `none`: 请求头中缺少“Referer”字段。

- `blocked`: “Referer”字段出现在请求头中, 但其值已被防火墙或代理服务器删除; 这些值是不以“http://”或“https://”开头的字符串。

- `server_names`: 服务器名称“Referer”请求头字段包含一个服务器名称。

- 任意字符串

  定义服务器名称和可选的URI前缀。服务器名称的开头或结尾可以有“*”。在检查过程中, “Referer”字段中的服务器端口被忽略。

- 正则表达式

  第一个符号应该是“~”。应该注意的是, 表达式将与“http://”或“https://”之后开始的文本匹配。

除了本文这种基于`http_referer`判定非法请求的方式, 还有一种计算MD5的判定, 不过比较麻烦所以暂且不表。

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
[Module ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html)
{{< /admonition >}}

