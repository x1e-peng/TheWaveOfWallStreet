---
date: "2019-06-12T23:00:48+08:00"
draft: true
title: "使用nginx搭建一个简单具有反向代理功能的网站"
tags: ["Nginx", "反向代理与负载均衡"]
series: ["nginx"]
categories: ["Nginx"]
toc: true
---

## `Nginx`介绍
作为一个web开发者，`Nginx`是最常用的web容器。

### 官方介绍   
`nginx`是基于`rest`架构风格，
以统一资源描述符`URI`或者统一资源定位符`URL`作为沟通依据，通过`HTTP`协议提供各种网络服务的web服务器。

### 我的理解   
物理上：nginx是由`二进制文件`、`nginx.conf`配置文件、`access log`、`error log`四部分组成。   
逻辑上：nginx是由各种各样的`模块（Modules）`组成。模块里面又包含`指令（Directives）`、`变量（Varibles）`
这两个重要的成员。

## 先搭建一个静态web网站

### 官网下载源码包
nginx的开源官网为`nginx.org`。我们可以直接去`download`里面下载最新版本源码包，或者复制下载路径到服务器上`wget`即可。

### 解压并编译安装nginx
我下载的是`gz`包，首先我们使用`tar`命令解压。然后就是标准的编译三部曲。
`./configure`一般后面会加一些自定义参数，如自定义安装目录的`--prefix`，这些都是可以按照自己的需求自定义。
具体的话可以执行`./configure --help`自己查看，我这里就按照最简单的安装了。
```shell
➜  xp ll
 -rw-r--r--   1 xiepeng  staff   1.0M  4 23 21:58 nginx-1.16.0.tar.gz
 ➜  xp tar xf nginx-1.16.0.tar.gz
 ➜  xp ./configure --prefix=/Users/xiepeng/xp/nginx
 ➜  xp make && make install
```
安装完成后，我们启动nginx。
然后打开浏览器，输入localhost就可以看到一个nginx的欢迎页面了。

{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/nginx_welcome.png" width='400px'/>{{% /center %}}

### 编译时常见报错
这里还有一个值得注意的是，通常这里会出现一个报错。提示你Nginx编译时对`openssl`报错了。

```shell
➜  xp /bin/sh: line 2: ./config: No such file or directory
➜  xp make[1]: *** [/usr/local/ssl/.openssl/include/openssl/ssl.h] Error 127
➜  xp make[1]: Leaving directory '/usr/local/src/nginx-1.16.0'
➜  xp make: *** [build] Error 2
```
此时我们有两种方式来解决这个问题：
#### 方法一：不安装openssl

```shell
➜  xp ./configure --prefix=/Users/xiepeng/xp/nginx --with-http_ssl_module
```

#### 方法二：修改openssl对应路径
根据报错信息我们知道，出错是因为Nginx在编译时并不能在`/usr/local/ssl/.openssl/`这个目录找到对应的文件，
其实我们打开`/usr/local/ssl/`这个目录可以发现这个目录下是没有`.openssl`目录的，因此我们修改Nginx编译时对
`openssl`的路径选择就可以解决这个问题了。   
打开nginx源文件下的`/Users/xiepeng/xp/nginx-1.16.0/auto/lib/openssl/conf`文件：
```shell
CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
CORE_LIBS="$CORE_LIBS $NGX_LIBDL"
```
修改为：
```shell
CORE_INCS="$CORE_INCS $OPENSSL/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libssl.a"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libcrypto.a"
CORE_LIBS="$CORE_LIBS $NGX_LIBDL"
```

## 配置反向代理与负载均衡
我们已经搭好了一个静态资源网站，现在要做反向代理与负载均衡相关的配置。
配置比较简单，我这里就之间贴上配置文件代码了。

我这里配置的是将`localhost:8000`端口进来的流量，平均的转发到`8001`端口和`8002`端口。
并将`8001`端口和`8002`端口分别配置了一个`a.test.pub`和`b.test.pub`的域名。

### `nginx.conf`配置
```shell
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    gzip  on;
    upstream test {
        server 127.0.0.1:8001 weight=1;
        server 127.0.0.1:8002 weight=1;
    }
    server {
        listen       8000;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://test;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    include vhost/*.conf;
}
```

### `vhost/a.test.pub.conf`配置
```shell
server {
        listen       8001;
        server_name  a.test.pub;

        location / {
            root   html/a.test.pub;
            index  index.html index.htm;
        }
    }
```

### 访问情况
第一次访问：
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/nginx_a.png" width='400px'/>{{% /center %}}

第二次访问：
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/nginx_b.png" width='400px'/>{{% /center %}}
