
---
date: 2016-12-31T11:32:52+08:00
title: "_gozk_zookeeper_zk_gofatalerror:zookeeper_hfilenotfound"
description: ""
disqus_identifier: 1485833572830878706
slug: "_gozk_zookeeper_zk_go-fatal-error:-zookeeper_h-file-not-found"
source: "https://segmentfault.com/a/1190000007518559"
tags: 
- zookeeper 
- golang 
topics:
- 编程语言与开发
---

*这个错误查了好久，最后在medium上找到了答案，遂记于此！*

**系统：** mac os

**错误信息：**

> /launchpad.net/gozk/zookeeper/zk.go:15:10: fatal error: 'zookeeper.h'
> file not found

**solution：**\
文件`$GOPATH/src/launchpad.net/gozk/zookeeper/zk.go`中

    ...
    package zookeeper

    /*
    #cgo CFLAGS: -I/usr/include/c-client-src -I/usr/include/zookeeper
    #cgo LDFLAGS: -lzookeeper_mt
    ...

修改为

    ...
    package zookeeper

    /*
    #cgo CFLAGS: -I/usr/include/c-client-src -I/usr/include/zookeeper -I/usr/local/include/zookeeper
    #cgo LDFLAGS: /usr/local/lib/libzookeeper_mt.a
    ...

参考：[https://medium.com/@robiplus/...](https://medium.com/@robiplus/build-gozk-in-osx-el-capitan-and-something-abount-go-c-interactions-b0eb6447d085#.civaayyzl)

