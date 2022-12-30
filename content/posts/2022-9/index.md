---
title: "Nginx按请求速率限速"
date: 2022-04-05T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Nginx,Security]
series: [Security Config of Nginx]
series_weight: 1
featuredImagePreview: ""
summary: 使用漏桶算法 (leaky bucket), 限制单个 IP (或者其他的 key) 发送请求的速率, 超出指定速率后, Nginx 将直接拒绝更多的请求, 对防御 DDoS 有一定的帮助。
---

## 算法

### 漏桶算法

漏桶算法 [(leaky bucket)](https://en.wikipedia.org/wiki/Leaky_bucket) 算法思想如图所示: 

{{< image src="1.png" caption="leaky bucket" height="auto" width="70%">}}

一个形象的解释是: 

- 水 (请求) 从上方倒入水桶, 从水桶下方流出 (被处理) ; 
- 来不及流出的水存在水桶中 (缓冲), 以固定速率流出; 
- 水桶满后水溢出 (丢弃)。

总结一下: 缓存请求、匀速处理、多余的请求直接丢弃。

### 令牌桶算法

令牌桶 [(token bucket)](https://www.cse.wustl.edu/~jain/cis788-97/ftp/integrated_services/index.html#tokenbucket) 算法思想如图所示: 

{{< image src="2.png" caption="token bucket" height="auto" width="70%">}}

算法思想是: 

- 令牌以固定速率产生, 并缓存到令牌桶中; 
- 令牌桶放满时, 多余的令牌被丢弃; 
- 请求要消耗等比例的令牌才能被处理; 
- 令牌不够时, 请求被缓存。

### 漏桶算法与令牌桶算法区别

相比漏桶算法, 令牌桶算法不同之处在于它不但有一只“桶”, 还有个队列, 这个桶是用来存放令牌的, 队列才是用来存放请求的。

从作用上来说, 漏桶和令牌桶算法最明显的区别就是是否允许**突发流量 (burst) **的处理, 漏桶算法能够**强行限制数据的实时传输 (处理) 速率**, 对突发流量不做额外处理; 而令牌桶算法能够在**限制数据的平均传输速率的同时允许某种程度的突发传输**。

Nginx按请求速率限速模块使用的是**漏桶算法**, 即能够强行保证请求的实时处理速度不会超过设置的阈值。

## Nginx限速模块

### 按连接数限速

按连接数限速, 是指限制单个IP (或者其他的key) 同时发起的连接数, 超出这个限制后, Nginx将直接拒绝更多的连接。官方文档: [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)

### 按请求速率限速

按请求速率限速, 是指限制单个IP (或者其他的key) 发送请求的速率, 超出指定速率后, Nginx将直接拒绝更多的请求。官方文档: [Module ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
    ...
    server {
    ...
        location / {
            limit_req zone=mylimit burst=4 nodelay; 
            ...
        }
    }
    ...
}
```

使用`limit_req_zone`关键字, 我们定义了一个名为`mylimit`大小为10MB的共享内存区域`zone`, 用来存放限速相关的统计信息, 限速的`key`值为二进制的IP地址`$binary_remote_addr`, 限速上限`rate`为2r/s; 接着我们使用`limit_req`关键字将上述规则作用到根目录上。`burst`和`nodelay`的作用稍后解释。

使用上述规则, 对于根目录的访问, 单个IP的访问速度被限制在了“2请求/秒”, 超过这个限制的访问将直接被Nginx拒绝。

{{< admonition type=quote title="$binary_remote_addr解释" open=true >}}
客户端IP地址用作密钥。请注意, 这里使用的不是`$remote_addr`, 而是`$binary_remote_addr`变量。`$binary_remote_addr`变量的大小对于IPv4地址始终为4字节, 对于IPv6地址始终为16字节。存储状态在32位平台上始终占64字节, 在64位平台上始终占128字节。一个兆字节的区域可以保存大约16000个64字节的状态或大约8000个128字节的状态。

如果区域存储耗尽, 则删除最近使用最少的状态。如果在此之后仍无法创建新状态, 则请求会因错误而终止。
{{< /admonition >}}

## 实验1, 毫秒级统计

我们有如下配置: 

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
    ...
    server {
    ...
        location / {
            limit_req zone=mylimit;
            ...
        }
    }
    ...
}
```

上述规则限制了每个IP访问的速度为2r/s, 并将该规则作用于根目录。如果单个IP在非常短的时间内并发发送多个请求, 结果会怎样呢? 

```
# 单个IP 10ms内并发发送6个请求
send 6 requests in parallel, time cost: 2 ms
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 200 OK
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 503 Service Temporarily Unavailable
end, total time cost: 461 ms
```

我们使用单个IP在10ms内发并发送了6个请求, 只有1个成功, 剩下的5个都被拒绝。我们设置的速度是2r/s, 为什么只有1个成功呢? 这是因为Nginx的限流统计是**基于毫秒**的, 我们设置的速度是2r/s, 转换一下就是500ms内单个IP只允许通过1个请求, 从501ms开始才允许通过第二个请求。

## 实验2, burst缓存处理突发请求

实验1我们看到, 我们短时间内发送了大量请求, Nginx按照毫秒级精度统计, 超出限制的请求直接拒绝。这在实际场景中未免过于苛刻, 真实网络环境中请求到来不是匀速的, 很可能有请求“突发”的情况, 也就是“一股子一股子”的。Nginx考虑到了这种情况, 可以通过`burst`关键字开启对突发请求的缓存处理, 而不是直接拒绝。

来看我们的配置: 

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
    ...
    server {
    ...
        location / {
            limit_req zone=mylimit burst=4;
            ...
        }
    }
    ...
}
```

我们加入了`burst=4`, 意思是每个key (此处是每个IP) 最多允许4个突发请求的到来。如果单个IP在10ms内发送6个请求, 结果会怎样呢? 

```
# 单个IP 10ms内发送6个请求, 设置burst
send 6 requests in parallel, time cost: 2 ms
HTTP/1.1 200 OK
HTTP/1.1 503 Service Temporarily Unavailable
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
HTTP/1.1 200 OK
end, total time cost: 2437 ms
```

相比实验1成功数增加了4个, 这个我们设置的`burst`数目是一致的。

具体处理流程是: 1个请求被立即处理, 4个请求被放到`burst`队列里, 另外一个请求被拒绝。通过`burst`参数, 使得Nginx限流具备了缓存处理突发流量的能力。

但是请注意: `burst`的作用是让多余的请求可以先放到队列里, 慢慢处理。如果不加`nodelay`参数, 队列里的请求**不会立即处理**, 而是按照`rate`设置的速度, 以毫秒级精确的速度慢慢处理。

## 实验3, nodelay无延迟排队

实验2中我们看到, 通过设置`burst`参数, 我们可以允许Nginx缓存处理一定程度的突发, 多余的请求可以先放到队列里, 慢慢处理, 这起到了平滑流量的作用。

但是如果队列设置的比较大, 请求排队的时间就会比较长, 用户角度看来就是RT变长了, 这对用户很不友好。有什么解决办法呢? `nodelay`参数允许请求在排队的时候就**立即被处理**, 也就是说只要请求能够进入`burst`队列, 就会立即被后台worker处理。这意味着`burst`设置了`nodelay`时, 系统瞬间的QPS可能会超过`rate`设置的阈值。另外, `nodelay`参数要跟`burst`一起使用才有作用。

延续实验2的配置, 我们加入`nodelay`选项: 

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=2r/s;
    ...
    server {
    ...
        location / {
            limit_req zone=mylimit burst=4 nodelay; 
            ...
        }
    }
    ...
}
```

单个IP在10ms内并发发送6个请求, 结果如下: 

```
# 单个IP 10ms内发送6个请求
实验3, 设置burst和nodelay         |  实验2, 只设置burst
send 6 requests, time cost: 4 ms |  time cost: 2 ms
HTTP/1.1 200 OK                  |  HTTP/1.1 200 OK
HTTP/1.1 200 OK                  |  HTTP/1.1 503 ...
HTTP/1.1 200 OK                  |  HTTP/1.1 200 OK
HTTP/1.1 200 OK                  |  HTTP/1.1 200 OK
HTTP/1.1 503 ...                 |  HTTP/1.1 200 OK
HTTP/1.1 200 OK                  |  HTTP/1.1 200 OK
total time cost: 465 ms          |  total time cost: 2437 ms
```

跟实验2相比, 请求成功率没变化, 但是**总体耗时变短了**。这怎么解释呢? 

- 实验2中, 有4个请求被放到burst队列当中, 工作进程每隔500ms(rate=2r/s)取一个请求进行处理, 最后一个请求要排队2s才会被处理。

- 实验3中, 请求放入队列跟实验2是一样的, 但不同的是, 队列中的请求同时具有了被处理的资格, 所以实验3中的5个请求可以说是同时开始被处理的, 花费时间自然变短了。

但是请注意, 虽然设置burst和nodelay能够降低突发请求的处理时间, 但是长期来看并不会提高吞吐量的上限, 长期吞吐量的上限是由rate决定的, 因为nodelay只能保证burst的请求被立即处理, 但Nginx会限制队列元素释放的速度, 就像是限制了令牌桶中令牌产生的速度。

看到这里你可能会问, 加入了nodelay参数之后的限速算法, 到底算是哪一个“桶”, 是漏桶算法还是令牌桶算法? 当然还算是漏桶算法。

考虑一种情况, 令牌桶算法的token为耗尽时会怎么做呢? 由于它有一个请求队列, 所以会把接下来的请求缓存下来, 缓存多少受限于队列大小。但此时缓存这些请求还有意义吗? 如果server已经过载, 缓存队列越来越长, RT越来越高, 即使过了很久请求被处理了, 对用户来说也没什么价值了。所以当token不够用时, 最明智的做法就是直接拒绝用户的请求, 这就成了漏桶算法。

## 两级限速

为了说明两阶段速率限制, 我们将Nginx配置为通过每秒5个请求 (r/s) 的速率限制来保护网站。假设该网站通常每页有4-6个资源, 而且资源从不超过12个。该配置允许最多12个请求的突发, 其中前8个请求被毫不延迟地处理。在执行5r/s限制的8个额外请求后, 会增加延迟。超过12个请求后, 任何进一步的请求都会被拒绝。

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=5r/s;
    ...
    server {
    ...
        location / {
            limit_req zone=mylimit burst=12 delay=8;
            ...
        }
    }
    ...
}
```

`delay`参数定义在突发大小内限制 (延迟) 过多请求以符合定义的速率限制的点。有了这种配置, 以8r/s的速度发出连续请求流的客户端会经历以下行为: 

{{< image src="3.png" caption="Two-Stage Rate Limiting" height="auto" width="100%">}}

前8个请求 (延迟值) 由Nginx无延迟代理。接下来的4个请求 (突发延迟) 被延迟, 以便不超过规定的5r/s速率。接下来的3个请求被拒绝, 因为已超过总突发大小。随后的请求被延迟。

## 限速白名单

```nginx
http {
    ...
    geo $limit {
        default 1;
        10.0.0.0/8 0;
        192.168.0.0/24 0;
    }
    
    map $limit $limit_key {
        0 "";
        1 $binary_remote_addr;
    }
    
    limit_req_zone $limit_key zone=mylimit:10m rate=5r/s;
    
    server {
    ...
        location / {
            limit_req zone=mylimit burst=10 nodelay;
            ...
        }
    }
    ...
}
```

本例同时使用`geo`和`map`指令。`geo`令白名单IP地址的`$limit`值为0, 使其他地址的`$limit`值默认为1。然后, 使用`map`将这些值转换为`key`: 

- 如果`$limit`为0, `$limit_key`设置为空字符串。
- 如果`$limit`为1, `$limit_key`设置为二进制格式的客户端IP地址。

当`limit_req_zone`目录 (key) 的第一个参数为空字符串时, 将不应用该限制, 因此允许列出的IP地址 (在`10.0.0.0/8`和`192.168.0.0/24`子网中) 不受限制。所有其他IP地址限制为每秒5个请求。

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [Nginx限速模块初探](https://www.cnblogs.com/CarpenterLee/p/8084533.html)
2. [Rate Limiting with NGINX and NGINX Plus](https://www.nginx.com/blog/rate-limiting-nginx/)
3. [Module ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
4. [Module ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)
{{< /admonition >}}