
---
date: 2016-12-31T11:32:26+08:00
title: "Go1_8rc3源代码学习:从源代码编译go"
description: ""
disqus_identifier: 1485833546700941070
slug: "Go-1_8rc3-yuan-dai-ma-xue-xi-:cong-yuan-dai-ma-bian-yi--go"
source: "https://segmentfault.com/a/1190000008228731"
tags: 
- golang 
topics:
- 编程语言与开发
---

前言
====

go 官方文档 [Installing Go from
source](https://golang.org/doc/install/source)详细说明了如何从 GitHub
源代码编译和安装 go

下载源代码
==========

    # git clone https://github.com/golang/go

构建
====

    # export GOROOT_BOOTSTRAP=/usr/local/go
    # src/all.bash

注：设置环境变量 GOROOT\_BOOTSTRAP

总结
====

