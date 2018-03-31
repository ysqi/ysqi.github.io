
---
date: 2016-12-31T11:32:34+08:00
title: "搭建golang版ss服务"
description: ""
disqus_identifier: 1485833554466258957
slug: "da-jian-golangban-ssfu-wu"
source: "https://segmentfault.com/a/1190000008050152"
tags: 
- golang 
- shadowsocks 
categories:
- 编程语言与开发
---

搭建golang版ss服务
==================

第一步：安装 golang
-------------------

    cd /usr/local     # golang安装到此路径下

    wget https://storage.googleapis.com/golang/go1.6.3.linux-amd64.tar.gz

    tar -xvf go1.6.3linux-amd64.tar.gz

    vim /etc/profile # 设置系统 GOROOT 并添加环境变量 

        export GOROOT=/usr/local/go

        export PATH=$PATH:$GOROOT/bin

    source /etc/profile     # 重载此文件

    cd ~     # 到家目录下面设置自己的GOPATH 这样不会污染其他用户的
    vim .bash_profile
        export GOPATH=$HOME/shadowsocks
    source .bash_profile

第二步：搭建 ss 服务
--------------------

    go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server

    cd shadowsocks/

    ls
        bin pkg src

    cd bin/

    shadowsocks-server -h # 查看帮助

    vim config.json # 编写配置文件
        {
            "server":"127.0.0.1",
            "server_port":8388,
            "local_port":1080,
            "password":"xxxx",
            "method": "aes-128-cfb-auth",
            "timeout":600
        }

    ./shadowsocks-server > log & # 在后台运行ss服务

客户端设置如下:

具体如何使用 可以看文档
[https://github.com/shadowsock...](https://github.com/shadowsocks/shadowsocks-go)

