
---
date: 2017-05-24T09:17:35+08:00
title: "Caddy新兴的web服务器caddy"
description: ""
disqus_identifier: 1495588655699071748
slug: "xin-xing-de-webfu-wu-qi-caddy"
source: "https://segmentfault.com/a/1190000008722666"
tags: 
- caddy 
- golang 
- nginx 
categories:
- 编程语言与开发
---

[caddy](https://caddyserver.com/) 是一个像 Apache, nginx, 或 lighttpd
的web服务器。\
你要问nginx已经很好了，为什么要用caddy呢?
我觉得caddy最大的特点是用起来简单，\
然后呢，它还有下面这些开箱即用的特性:

-   `HTTP/2` 全自动支持HTTP/2协议，无需任何配置。

-   `Auto HTTPS` Caddy 使用 Let's Encrypt
    让你的站点全自动变成全站HTTPS，无需任何配置。当然你想使用自己的证书也是可以的。

-   `Multi-core` 因为caddy是golang写的，所以当然可以合理使用多核啦。

-   `IPv6` 完全支持IPv6环境.

-   `WebSockets` Caddy 对WebSockets有很好的支持.

-   `Markdown` 自动把md转成 HTML
    ，当然，我后续要给大家介绍更强大的hugo来干这个事情.

-   `Logging` Caddy 对log格式的定义很容易，更好的满足你日志收集的需求。

-   `Easy Deployment`
    得益于go的特性，caddy只是一个小小的二进制文件，没有依赖，很好部署。

那么在什么场景下适合尝试使用`caddy`呢，我推荐从以下场景开始：

-   作为静态页面的webserver

-   转发 fastcgi 请求到 php-fpm
    服务，比如替换apache或nginx作为wordpress的server

-   反向代理，管理多个站点

-   微服务的 API gateway ，我会专门写一篇文章。

-   有些在nginx上难以开发的需求，为caddy写插件太方便了。

入门
----

### 安装caddy

1.  [下载](https://caddyserver.com/download) caddy

2.  把caddy放到系统的PATH中，让其可以直接执行。比如Linux中一般习惯放到
    `/usr/local/bin`

### 简单测试

1.  找一个做测试的临时目录，生成一个测试主页。`echo "hello world">index.html`

2.  执行 `caddy`

3.  在另一个终端 `curl localhost:2015` 或在浏览器访问
    ([http://localhost:2015)](http://localhost:2015))

### Caddyfile

caddy的一个特色就是配置简单，nginx的配置文件群已经越看越晕了。我们来试试：

在当前目录创建这样一个叫`Caddyfile`的文件：

    localhost:2020
    gzip

这次，我们改变了端口，并且启用了gzip自动压缩数据。运行`caddy`，去你指定的地址看看吧。

说一句，caddy的潜规则是找当前目录叫`Caddyfile`的文件，你也可以用参数指定文件和路径。

更专业一点
----------

我们随便说点高级功能，其实caddy的[文档](https://caddyserver.com/docs)挺不错的，看文档就可以了解各种功能。

### 自动 HTTPS

如果你满足这些条件，你用caddy启动的应用将自动获得HTTPS，不用你买证书了，这都是`Let's Encrypt`的功劳。

-   host 那里要填一个域名，不能是 localhost 或 IP

-   不要用冒号手动指定端口

-   不要在域名前手动声明http

-   没在配置里关掉TLS 或者声明用自己的证书但是还没配好

-   caddy 有权限绑定 80 和 443 端口

前边都能懂，说下最后一条。在init文件夹的启动配置教程里都有，一般建议你用www-data用户启动服务，\
你不是root但是Linux依然可以让你绑定80端口，只需要执行`setcap cap_net_bind_service=+ep caddy`
。\
具体看文档吧。

### 多站点

你可能想，之前用nginx主要是为了支持多站点，caddy当然也是可以的，你只需要配置若干域名，\
把每个域名的配置写在后边的大括号配置块里就行了。下一个例子里就有。

### PHP or Wordpress

据说全世界四分之一的站点都是wordpress搭建的，而PHP公认是世界上最好的语言。\
caddy还没有完全支持unix socket通讯呢，赶忙先把PHP支持了再说。

这是我自己博客的配置片段，我的荒芜的非技术博客依然用的wordpress。\
`timeouts`关键字是我摸索出来的，官方示例没有，不设置这个国内升级插件什么的根本成功不了。\
`tls`其实用默认值是可以的，但是后台会有一堆落后的搜索引擎和爬虫报错，于是我调低了一点。\
另外我还把www定向到了裸域名，大家一般都这样做，或者反过来。

    xiafeng.net {
        root /data/xiafeng/public
        timeouts 10m
        gzip
        tls {
            protocols tls1.0 tls1.2
        }
        fastcgi / unix:/var/run/php/php7.0-fpm.sock php
        rewrite {
            if {path} not_match ^\/wp-admin
            to {path} {path}/ /index.php?_url={uri}
        }
    }

    www.xiafeng.net {
        redir https://xiafeng.net
    }

### 开机启动

因为大部分发行版目前还没办法直接安装caddy，开机启动可能需要你自己动手啦。

在你下载的压缩包中有一个init文件夹，里边有Mac,Linux,FreeBSD的开机启动配置帮助，\
还有示例脚本，可以根据你的要求再DIY一下。

预告
----

作为入门就先介绍这么多，我接下来的博客将会写一些好玩的或专业的caddy的用法。敬请期待。

