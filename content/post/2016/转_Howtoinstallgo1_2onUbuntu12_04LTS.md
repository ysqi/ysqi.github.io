
---
date: 2016-12-31T11:34:57+08:00
title: "Howtoinstallgo1_2onUbuntu12_04LTS"
description: ""
disqus_identifier: 1485833697334112205
slug: "How-to-install-go-1_2-on-Ubuntu-12_04-LTS"
source: "https://segmentfault.com/a/1190000000486267"
tags: 
- golang 
topics:
- 编程语言与开发
---

Download archive tar.gz from
[here](http://code.google.com/p/go/downloads/list?q=OpSys-FreeBSD%20OR%20OpSys-Linux%20OR%20OpSys-OSX%20Type-Archive)

    sudo tar -C /usr/local -xzf go1.2.1.linux-amd64.tar.gz

    export PATH=$PATH:/usr/local/go/bin

    export GOROOT=/usr/local/go

    go version 

    `go version go1.2.1 linux/amd64`

