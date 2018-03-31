
---
date: 2016-12-31T11:33:22+08:00
title: "glide使用"
description: ""
disqus_identifier: 1485833602508359285
slug: "glideshi-yong"
source: "https://segmentfault.com/a/1190000005929355"
tags: 
- glide 
- golang 
categories:
- 编程语言与开发
---

##### github:

<https://github.com/Masterminds/glide>

#### document:

<http://glide.readthedocs.io/en/stable/?badge=stable>

##### golang环境设置

    设置环境变量 使用vendor目录
    GO15VENDOREXPERIMENT=1

##### 安装

    获取
    $ go get github.com/Masterminds/glide
    进入目录
    $ cd github.com/Masterminds/glide
    编译
    $ make build
    $ go build -o glide -ldflags "-X main.version=v0.11.0" glide.go

##### 使用

    # 创建glide.yaml文件 提示选择N(选择Y 是重新配置文件需手动指定)
    $ glide create
    #打开配置文件
    $ open glide.yaml                         
    #使用glide获取包会自动写入glide.yaml文件中
    $ glide get github.com/Masterminds/cookoo
    # 安装glide.yaml所需的包
    $ glide install
    # 项目编译
    $ go build
    # glide更新包
    $ glide up                                

\*注意
======

可以指定下载包的地址，版本号，下载方式\
例如(golang包需要指定下载地址):

    - package: golang.org/x/net/context
    repo:    git@github.com:golang/net.git
    vcs:     git

上述的编写方式用空格做缩进，所有key对齐。

完整的配置文件格式
==================

    package: github.com/Masterminds/glide
    homepage: https://masterminds.github.io/glide
    license: MIT
    owners:
    - name: Matt Butcher
    email: technosophos@gmail.com
    homepage: http://technosophos.com
    - name: Matt Farina
    email: matt@mattfarina.com
    homepage: https://www.mattfarina.com
    ignore:
    - appengine
    excludeDirs:
    - node_modules
    import:
    - package: gopkg.in/yaml.v2
    - package: github.com/Masterminds/vcs
    version: ^1.2.0
    repo:    git@github.com:Masterminds/vcs
    vcs:     git
    - package: github.com/codegangsta/cli
    - package: github.com/Masterminds/semver
    version: ^1.0.0
    testImport:
    - package: github.com/arschles/assert

#### 版本号指定(version字段)

    =: equal (aliased to no operator)
    !=: not equal
    >: greater than
    <: less than
    >=: greater than or equal to
    <=: less than or equal to

    1.2 - 1.4.5 which is equivalent to >= 1.2, <= 1.4.5
    2.3.4 - 4.5 which is equivalent to >= 2.3.4, <= 4.5
    1.2.x is equivalent to >= 1.2.0, < 1.3.0

    >= 1.2.x is equivalent to >= 1.2.0
    <= 2.x is equivalent to < 3
    * is equivalent to >= 0.0.0

    ~1.2.3 is equivalent to >= 1.2.3, < 1.3.0
    ~1 is equivalent to >= 1, < 2
    ~2.3 is equivalent to >= 2.3, < 2.4
    ~1.2.x is equivalent to >= 1.2.0, < 1.3.0
    ~1.x is equivalent to >= 1, < 2

    ^1.2.3 is equivalent to >= 1.2.3, < 2.0.0
    ^1.2.x is equivalent to >= 1.2.0, < 2.0.0
    ^2.3 is equivalent to >= 2.3, < 3
    ^2.x is equivalent to >= 2.0.0, < 3

'\*'指定版本报错，需要用'\*'指定的可以不填写

