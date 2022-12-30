---
title: "Nginx动态编译GeoIP2模块"
date: 2022-04-07T10:00:00+08:00
draft: false
authors: ["SenZyo"]
tags: [Linux,Nginx,Security]
series: [Security Config of Nginx]
series_weight: 3
featuredImagePreview: ""
summary: 动态编译 `ngx_http_geoip2_module`, 在 `/etc/nginx/nginx.conf` 中引用 `ngx_http_geoip2_module.so` 和 GeoLite2 免费数据库, 最终达到屏蔽来自海外连接, 且允许常见海外爬虫访问的效果。
---

## 依赖

### pcre和openssl

Nginx不支持[pcre2](https://github.com/PhilipHazel/pcre2), 另外Debian系Linux中, `pcre`和`openssl`的`lib`包名和其他系统不一样。

CentOS: 

```bash
sudo yum install pcre-devel openssl openssl-devel
```

Debian: 

```bash
sudo apt install libpcre3 libpcre3-dev libssl-dev
```

### MaxMind

```bash
sudo apt install libmaxminddb0 libmaxminddb-dev mmdb-bin
```

如果源仓库中没有`libmaxminddb0`、`libmaxminddb-dev`、`mmdb-bin`中的某个包, Ubuntu可以通过添加PPA来解决: 

```bash
sudo add-apt-repository ppa:maxmind/ppa
sudo apt-get update 
sudo apt install libmaxminddb0 libmaxminddb-dev mmdb-bin
```

但PPA方式在Debian上不太好用, 可以先安装[geoipupdate](https://github.com/maxmind/geoipupdate), `geoipupdate`包含了上面缺失的某个包。

#### GeoIP Update

安装`geoipupdate`, 推荐下载[Release](https://github.com/maxmind/geoipupdate/releases)中的`deb`包, 而不是`sudo apt install geoipupdate`安装, 因为Release中的版本高于Debian仓库。关于`geoipupdate`的相关配置, 本文后面的[geoipupdate配置](#geoipupdate配置)会讲到。

实在不行可以从[libmaxminddb](https://github.com/maxmind/libmaxminddb)的源码直接编译安装`libmaxminddb0`、`libmaxminddb-dev`、`mmdb-bin`这3个包。

## 编译

下载Nginx的源码包并解压: 

```bash
cd ~
wget http://nginx.org/download/nginx-1.22.1.tar.gz
tar -zxvf nginx-1.22.1.tar.gz
```

克隆仓库[ngx_http_geoip2_module.git](https://github.com/leev/ngx_http_geoip2_module.git)到`/usr/lib/nginx/modules/ngx_http_geoip2_module`: 

```bash
git clone https://github.com/leev/ngx_http_geoip2_module.git /usr/lib/nginx/modules/ngx_http_geoip2_module
```

编译前使用`nginx -V`命令先看一下已安装的Nginx的参数作为参考, 输出类似: 

```nginx
nginx version: nginx/1.18.0
built with OpenSSL 1.1.1n  15 Mar 2022
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module
```

`cd ~/nginx-1.22.1/`, 运行以下命令配置编译参数: 

```bash
./configure --prefix=/usr/share/nginx \
--conf-path=/etc/nginx/nginx.conf \
--http-log-path=/var/log/nginx/access.log \
--error-log-path=/var/log/nginx/error.log \
--lock-path=/var/lock/nginx.lock \
--pid-path=/run/nginx.pid \
--modules-path=/usr/lib/nginx/modules \
--http-client-body-temp-path=/var/lib/nginx/body \
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
--http-proxy-temp-path=/var/lib/nginx/proxy \
--http-scgi-temp-path=/var/lib/nginx/scgi \
--http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
--with-compat \
--with-debug \
--with-pcre-jit \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_realip_module \
--with-http_auth_request_module \
--with-http_v2_module \
--with-http_dav_module \
--with-http_slice_module \
--with-threads \
--with-http_addition_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_sub_module \
--with-stream \
--add-dynamic-module=/usr/lib/nginx/modules/ngx_http_geoip2_module
```

使用`make`命令编译, 编译完成的文件在`~/nginx-1.22.1/objs/`路径下。

{{< admonition type=warning title="不要make install" open=true >}}

应先备份旧版本Nginx以防万一: 

```bash
sudo mv /usr/sbin/nginx ~/nginx_$(date +%F)
```

再将编译好的新版本替换上去: 

```bash
sudo cp ~/nginx-1.22.1/objs/nginx /usr/sbin/
```

{{< /admonition >}}

看一下当前Nginx的参数`nginx -V`, 输出中应该已经包括`--add-dynamic-module=/usr/lib/nginx/modules/ngx_http_geoip2_module`

复制编译好的模块到指定位置, 以便后续在`nginx.conf`中引用: 

```bash
sudo cp ~/nginx-1.22.1/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules/
sudo cp ~/nginx-1.22.1/objs/ngx_stream_geoip2_module.so /usr/lib/nginx/modules/
```

## 配置

### geoipupdate配置

参考[官方文档](https://dev.maxmind.com/geoip/updating-databases?lang=en#2-obtain-geoipconf-with-account-information), 创建[License Keys](https://www.maxmind.com/en/accounts/current/license-key), 下载[GeoIP.conf](https://www.maxmind.com/en/accounts/current/license-key/GeoIP.conf)保存到`/etc/`下, 创建`crontab`定时任务运行`geoipupdate`, 比如: 

```bash
0 3 * * 1 flock -xn /tmp/geoipupdate.lock -c '/usr/bin/geoipupdate > /dev/null 2>&1 &'
```

{{< admonition type=tip title="geoipupdate安装路径" open=true >}}

安装路径不一定是官方说的`/usr/local/bin/geoipupdate`, 也可能是`/usr/bin/geoipupdate`。

{{< /admonition >}}

`geoipupdate`会将数据下载到`/usr/share/GeoIP/`路径下。

### Nginx配置

在`/etc/nginx/nginx.conf`中引用`ngx_http_geoip2_module.so`和GeoLite2免费数据库 (这里用的是`GeoLite2-Country.mmdb`), 最终达到屏蔽来自海外连接, 且允许常见海外爬虫访问的效果。

```nginx
...
load_module /usr/lib/nginx/modules/ngx_http_geoip2_module.so;
load_module /usr/lib/nginx/modules/ngx_stream_geoip2_module.so;
...
http {
    ...
    geoip2 /usr/share/GeoIP/GeoLite2-Country.mmdb {
        # 启用自动重载将使nginx在指定的时间间隔内检查数据库的修改时间, 并在数据库已更改时重新加载它。
        auto_reload 168h;
        $geoip2_metadata_country_build metadata last_check;
        $geoip2_data_country_code country iso_code;
    }
    map $geoip2_data_country_code $allowed_country {
        # 默认拒绝连接, 只允许来自中国的连接
        default no;
        CN yes;    
    }
    ...
    server {
        ...
        if ($allowed_country = no) {
           set $blocked A;
        }
        # 允许常见海外爬虫访问
        if ($http_user_agent !~* (bingbot|DuckDuckBot|Googlebot|YandexBot)) {
           set $blocked "${blocked}B";
        }
        # 来自海外的连接, 且不是允许的海外爬虫, 即拒绝连接
        if ($blocked = AB){
           return 444;
        }
        ...
    }
}
```

--------------------

{{< admonition type=quote title="参考链接" open=true >}}
1. [NGINX block by country and exclude bots](https://stackoverflow.com/questions/67545513/nginx-block-by-country-and-exclude-bots)
2. [ ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module)
{{< /admonition >}}

