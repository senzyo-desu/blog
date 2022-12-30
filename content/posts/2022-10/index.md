---
title: "Nginx按请求数拉黑IP"
date: 2022-04-06T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Bash,Linux,Nginx,Security]
series: [Security Config of Nginx]
series_weight: 2
featuredImagePreview: ""
summary: 通过 `crontab` 任务定时筛选访问日志 `access.log` 数据, 将访问量不正常的 IP (比如将单分钟内请求数超过阈值) 加入黑名单, 再由 Nginx 读取黑名单, 阻止其访问。
---

{{< admonition type=info title="替代方案" open=true >}}
也可以使用[fail2ban](https://www.fail2ban.org/)的`[nginx-limit-req]`功能。
{{< /admonition >}}

## 定时筛选访问日志

`/etc/nginx/nginx.conf`设置日志输出格式: 

```nginx
http {
    ...
    log_format main '[$time_iso8601] $remote_addr '
                    '$request_method $scheme://$host$request_uri $status '
                    '"$http_user_agent"';
    access_log /usr/share/nginx/logs/access.log main;
    error_log /dev/null;
    ...
}
```

`access.log`输出展示: 

```
[2022-12-18T17:16:01+08:00] 49.233.38.100 GET https://www.example.com/2022-24/ 200 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36 Edg/108.0.1462.46"
[2022-12-18T17:16:11+08:00] 49.233.38.100 OPTIONS https://twikoo.example.com/ 204 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36 Edg/108.0.1462.46"
[2022-12-18T17:17:22+08:00] 49.233.38.101 OPTIONS https://twikoo.example.com/ 204 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36 Edg/108.0.1462.46"
[2022-12-18T17:18:33+08:00] 49.233.38.102 POST https://twikoo.example.com/ 200 "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36 Edg/108.0.1462.46"
```

创建`/usr/share/nginx/conf/blockip.sh`文件筛选访问日志`access.log`数据, 文件内容如下: 

```bash
#!/bin/bash
awk -v ago=$(date '+[%FT%T]' -d '-5 minutes') '$1 > ago' /usr/share/nginx/logs/access.log \
| awk '{print $1,$2}' \
| cut -d ':' -f 1,2,4 \
| sort | uniq -c | sort -rn \
| awk '{if($1>400)print "deny " $3 ";"}' >> /usr/share/nginx/conf/blockip.conf
sort -u /usr/share/nginx/conf/blockip.conf -o /usr/share/nginx/conf/blockip.conf
```

授予可执行权限: 

```bash
chmod +x /usr/share/nginx/conf/blockip.sh
```

`crontab -e`编辑定时运行任务, 添加: 

```bash
*/5 * * * * flock -xn /tmp/blockip.lock -c '/usr/share/nginx/conf/blockip.sh > /dev/null 2>&1 &'
```

`crontab -l`查看定时任务, 检查一下。

### blockip.sh脚本解释

`date -d '-5 minutes'`表示输出5分钟前的时间。

`date +`指定时间格式, 详见[Linux date命令的参数格式](../2022-11/), 格式必须与对应字段 (这里是`/usr/share/nginx/logs/access.log`的第一个列字段) 相同。

5分钟前的时间被赋值给一个用户定义变量`ago`。

所以`awk -v ago=$(date '+[%FT%T]' -d '-5 minutes') '$1 > ago' /usr/share/nginx/logs/access.log`就是输出`access.log`中5分钟前至现在的记录的第一列字段`$1`, 即时间字段。

`awk '{print $1,$2}'`将结果的第1列和第2列字段输出, 即`[$time_iso8601]`和`$remote_addr`: 

```
[2022-12-18T17:16:01+08:00] 49.233.38.100
[2022-12-18T17:16:11+08:00] 49.233.38.100
[2022-12-18T17:17:22+08:00] 49.233.38.101
[2022-12-18T17:18:33+08:00] 49.233.38.102
```

`cut -d ':' -f 1,2,4`是以`:`为间隔, 切出第1、2、4字段, 相当于丢弃了第3字段`秒数+08`, 输出: 

```
[2022-12-18T17:16:00] 49.233.38.100
[2022-12-18T17:16:00] 49.233.38.100
[2022-12-18T17:17:00] 49.233.38.101
[2022-12-18T17:18:00] 49.233.38.102
```

`sort`将结果排序, `uniq -c`在每列旁边显示该行重复出现的次数, `sort -rn`依照数值的大小倒序排序, 输出: 

```
2 [2022-12-18T17:16:00] 49.233.38.100
1 [2022-12-18T17:17:00] 49.233.38.101
1 [2022-12-18T17:18:00] 49.233.38.102
```

`awk '{if($1>400)print "deny " $3 ";"}' >> /usr/share/nginx/conf/blockip.conf`表示当结果的`$1`字段 (即单分钟内请求数) 大于400时, 将`deny $3 (即IP) `追加到文件`blockip.conf`中。

`sort -u`将`/usr/share/nginx/conf/blockip.conf`中的所有唯一值输出 (即去除重复), `-o`表示输出到文件` /usr/share/nginx/conf/blockip.conf`中。

## Nginx.conf

```nginx
http {
    ...
    include /usr/share/nginx/conf/blockip.conf;
    ...
}
```

在`/etc/nginx/nginx.conf`中引入黑名单文件, `nginx -s reload`重载Nginx配置。
