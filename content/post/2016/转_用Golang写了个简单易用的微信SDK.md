
---
date: 2016-12-31T11:33:04+08:00
title: "用Golang写了个简单易用的微信SDK"
description: ""
disqus_identifier: 1485833584078282565
slug: "yong-Golangxie-le-ge-jian-chan-yi-yong-de-wei-xin-SDK"
source: "https://segmentfault.com/a/1190000006933953"
tags: 
- 微信公众平台 
- sdk 
- 微信 
- golang 
categories:
- 编程语言与开发
---

WeChat SDK for Go
=================

使用Golang开发的微信SDK，简单、易用。

项目地址：<https://github.com/silenceper/wechat>

文档地址：[DOCS](https://github.com/silenceper/wechat/blob/master/README.md)

快速开始
--------

以下是一个处理消息接收以及回复的例子：

    //配置微信参数
    config := &wechat.Config{
        AppID:          "xxxx",
        AppSecret:      "xxxx",
        Token:          "xxxx",
        EncodingAESKey: "xxxx",
        Cache:          memCache
    }
    wc := wechat.NewWechat(config)

    // 传入request和responseWriter
    server := wc.GetServer(request, responseWriter)
    server.SetMessageHandler(func(msg message.MixMessage) *message.Reply {

        //回复消息：演示回复用户发送的消息
        text := message.NewText(msg.Content)
        return &message.Reply{message.MsgText, text}
    })

    server.Serve()
    server.Send()

完整代码：examples/http/http.go

#### 和主流框架配合使用

主要是request和responseWriter在不同框架中获取方式可能不一样：

-   Beego: ./examples/beego/beego.go

-   Gin Framework: ./examples/gin/gin.go

基本配置
--------

    memcache := cache.NewMemcache("127.0.0.1:11211")

    wcConfig := &wechat.Config{
        AppID:          cfg.AppID,
        AppSecret:      cfg.AppSecret,
        Token:          cfg.Token,
        EncodingAESKey: cfg.EncodingAESKey,//消息加解密时用到
        Cache:          memcache,
    }

**Cache 设置**

Cache主要用来保存全局access\_token以及js-sdk中的ticket：\
默认采用memcache存储。当然也可以直接实现`cache/cache.go`中的接口

基本API使用
-----------

-   消息管理

    -   接收普通消息

    -   接收事件推送

    -   被动回复消息

        -   回复文本消息

        -   回复图片消息

        -   回复视频消息

        -   回复音乐消息

        -   回复图文消息

-   自定义菜单

    -   自定义菜单创建接口

    -   自定义菜单查询接口

    -   自定义菜单删除接口

    -   自定义菜单事件推送

    -   个性化菜单接口

        -   添加个性化菜单

        -   删除个性化菜单

        -   测试个性化菜单匹配结果

    -   获取公众号菜单配置

-   微信网页开发

    -   Oauth2 授权

        -   发起授权

        -   通过code换取access\_token

        -   拉取用户信息

        -   刷新access\_token

        -   检验access\_token是否有效

    -   获取js-sdk配置

-   素材管理

更多API使用请参考文档：\
[https://github.com/silenceper...](https://github.com/silenceper/wechat/blob/master/README.md)

