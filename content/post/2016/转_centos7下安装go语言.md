
---
date: 2016-12-31T11:34:01+08:00
title: "centos7下安装go语言"
description: ""
disqus_identifier: 1485833641929534428
slug: "centos-7-xia--an-zhuang-goyu-yan"
source: "https://segmentfault.com/a/1190000004223631"
tags: 
- linux 
- 开发环境配置 
- shell 
- golang 
categories:
- 编程语言与开发
---

1.下载 并且 安装 Go安装包

百度网盘上传了最新GO版本，供大家下载：<http://pan.baidu.com/s/1bjg9zg>

===========================================================\
在/usr/local下安装程序

    $ tar -xzf go1.5.2.linux-xxx.tar.gz -C /usr/local

在/etc/profile中添加系统范围的PATH环境变量。

    $ vi /etc/profile

export PATH=\$PATH:/usr/local/go/bin

如果你在/usr/local之外的自定义位置中安装了Go，你同样需要设置GOROOT环境变量来指向自定义的安装位置。

    $ vi /etc/profile

export GOPATH=/root/wwwgo\
export GOROOT=/usr/local/go

刷新环境变量

    $ source /etc/profile

检查Go语言的版本

    $ go version

go version go1.5.2 linux/amd64

