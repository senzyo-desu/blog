---
title: "Nginx禁止直接IP访问"
date: 2022-09-06T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Nginx,Security]
series: [Security Config of Nginx]
series_weight: 5
featuredImagePreview: ""
summary: 通过设置 `default_server` 和 `ssl_reject_handshake on` 来拒绝直接通过 IP 访问网站的请求。
---

## Nginx v1.19.4及后续版本

```nginx
...
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name 192.168.1.1;
    return 444;
}
server {
    listen 443 default_server;
    listen [::]:443 default_server;
    server_name 192.168.1.1;
    ssl_reject_handshake on;
}
...
```

值得一提的是, 相比`server_name`后接服务器的IP地址, `server_name  _;`更为常见, 虽然两种写法的作用效果相同, 但是[Nginx官方文档](http://nginx.org/en/docs/http/server_names.html#miscellaneous_names)中并不提倡使用`server_name  _;`, 并指出: 

{{< admonition type=quote title="原文" open=true >}}
There is nothing special about this name, it is just one of a myriad of invalid domain names which never intersect with any real name. Other invalid names like “--” and “!@#” may equally be used.
{{< /admonition >}}

## Nginx v1.19.4先前版本

Nginx v1.19.4先前版本不支持`ssl_reject_handshake`, 所以只能先指定证书, 再`return 444`。

```nginx
...
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name 192.168.1.1;
    return 444;
}
server {
    listen 443 default_server;
    listen [::]:443 default_server;
    server_name 192.168.1.1;
    ssl_certificate /etc/nginx/ssl/default.crt;
    ssl_certificate_key /etc/nginx/ssl/default.key;
    return 444;
}
...
```

## 不推荐使用if

当服务器只托管一个网站时, 这可能是最简单的解决方案, 但并不优雅, 因为不方便后续扩展网站。

```nginx
...
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name example.com;
    if ($host != example.com) {
        return 444;
    }
}
server {
    listen 443 default_server;
    listen [::]:443 default_server;
    server_name example.com;
    if ($host != example.com) {
        return 444;
    }
}
...
```

{{< admonition type=warning title="" open=true >}}
它将阻止访问其他子域, 比如`www.example.com`, `blog.example.com`。
{{< /admonition >}}

如果要支持更多子域, 可以使用正则表达式来执行此操作, 比如这适用于`example.com`和`www.example.com`: 

```nginx
    if ($host !~* ^(www\.)?example.com$) {
        return 444;
    }
```

显而易见, 如果有更多的子域, 就与要更复杂的正则表达式来匹配, 这很容易出错, 所以还是`ssl_reject_handshake`方案好。