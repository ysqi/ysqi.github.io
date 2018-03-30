
---
date: 2016-12-31T11:35:00+08:00
title: "heartbleeder自动检测OpenSSL心脏出血漏洞(附修复指南)"
description: ""
disqus_identifier: 1485833700405788344
slug: "heartbleeder-zi-dong-jian-ce--OpenSSL-xin-zang-chu-xie-lou-dong--(fu-xiu-fu-zhi-na-)"
source: "https://segmentfault.com/a/1190000000461002"
tags: 
- 开源项目介绍 
- golang 
- 安全 
- 内存 
- openssl 
topics:
- 编程语言与开发
---

[heartbleeder](https://github.com/titanous/heartbleeder)
可以探测你的服务器是否存在 OpenSSL
[CVE-2014-0160](https://www.openssl.org/news/secadv_20140407.txt) 漏洞
（心脏出血漏洞）。

什么是心脏出血漏洞？
--------------------

CVE-2014-0160，心脏出血漏洞，是一个非常严重的 OpenSSL
漏洞。这个漏洞使得攻击者可以从存在漏洞的服务器上读取64KB大小的内存信息。这些信息中可能包含非常敏感的信息，包括用户请求、密码甚至证书的私钥。

据称，已经有攻击者在某宝上尝试使用漏洞读取数据，在读取200次后，获取了40多个用户名和7个密码。

如何使用 heartbleeder 检测心脏出血漏洞？
----------------------------------------

### 安装

可以在[gobuild.io](https://gobuild.io/download/github.com/titanous/heartbleeder)下载编译好的二进制文件的压缩包。包括Windows、Linux、MacOSX。

由于服务器操作系统最常用的是Linux，因此这里提供一下下载Linux二进制压缩包的命令：

Linux(amd64)

    wget http://gobuild.io/github.com/titanous/heartbleeder/master/linux/amd64 -O output.zip

Linux(i386)

    wget http://gobuild.io/github.com/titanous/heartbleeder/master/linux/386 -O output.zip

下载后解压缩即可。

也可以自行编译安装（Go版本需在1.2以上）， 使用如下命令：

    go get github.com/titanous/heartbleeder

二进制文件会放置在 `$GOPATH/bin/heartbleeder`。

### 使用

    $ heartbleeder example.com
    INSECURE - example.com:443 has the heartbeat extension enabled and is vulnerable

Postgres 默认在 5432 端口使用
OpenSSL，如果你使用Postgres服务器，则需使用如下命令：

    $ heartbleeder -pg example.com
    SECURE - example:5432 does not have the heartbeat extension enabled

如何手工检测心脏出血漏洞
------------------------

如果不方便安装heartbleeder，或者不放心自动检测的结果，也可以手动检测。

首先判断服务器上的Openssl版本是否是有漏洞的版本。目前有漏洞的版本有：
`1.0.1-1.0.1f`（包含1.0.1f）以及
`1.0.2-beta`。你可以使用如下的命令查看服务器上的当前版本：

    openssl version

接着你需要判断是否开启了心跳扩展：

    openssl s_client -connect 你的网站:443 -tlsextdebug 2>&1| grep 'TLS server extension "heartbeat" (id=15), len=1'

如果以上两个条件你都满足的话，很遗憾，你的服务器受此漏洞影响，需要尽快修复。

如何修复
--------

1.  将受影响的服务器下线，避免它继续泄露敏感信息。
2.  停止旧版的 openssl 服务，升级 openssl 到新版本，并重新启动。
3.  生成新密钥。（因为攻击者可能通过漏洞获取私钥。）将新密钥提交给你的CA，获得新的认证之后在服务器上安装新密钥。
4.  服务器上线。
5.  撤销旧认证。
6.  撤销现有的会话cookies。
7.  要求用户修改密码。

------------------------------------------------------------------------

编撰 [SegmentFault](http://sf.gg)

