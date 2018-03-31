
---
date: 2016-12-31T11:34:22+08:00
title: "使用Go语言框架进行web开发笔记"
description: ""
disqus_identifier: 1485833662824853468
slug: "shi-yong-Goyu-yan-kuang-jia-jin-hang-webkai-fa-bi-ji"
source: "https://segmentfault.com/a/1190000003491898"
tags: 
- goroutine 
- martini 
- golang 
categories:
- 编程语言与开发
---

> 最近需要用Instagram的api抓取其用户的图片，由于需要用oauth2验证,
> 所以应用必须包含一个web界面。设想能够实时返回下载数量，所以用websocket。还有需要考虑到效率问题，综合以上几点，想用一门语言开发的话，最终选择用golang进行开发，node的回调实在不喜欢。

前言
----

关于golang的web开发有不少框架，例如 martini, gin, revel，gorilla等。
之前玩过`revel`，感觉封装的太多了，作为一个小应用不需要这么复杂，而且google得到结果是revel的效率相对较差。`gin`的benchmark显示效率是martini的40倍，但是gin比较新所以他的的生态圈相对较少。最终选择了`martini`,
有很多middleware可以选择，其中就包括了websocket，并且背后用的是gorilla
websocket这个包。

界面和功能
----------

1.  一个跳转到Oauth2登陆授权页面的链接

2.  授权完成后，跳回服务的页面，此时获得了access\_token,
    就可以为所欲为了。全部的功能也都集中在这个页面，最终的界面如下图所示。

`点击连接`是用来打开websocket连接的。`开始发送数据`是开始把用户ID发给服务端，服务端调用api开始抓取图片。`停止`用于停止本次的抓取服务。`已完成数量`用于实时返回抓取的图片数量。

程序大致结构
------------

这里把`Jobs`, `goroutine #1`,
`#2`等作用在全局是为了在websocket断开后，下载还能继续执行。`websocket goroutine`是连接建立后的作用域，连接断开后这个goroutine就不存在了。`Jobs`,
`NextUrl`充当队列的角色。
`Done`的作用仅仅是计数。这里少写了两个全局变量，`Quit chan int`,
`IsPreparing bool`, 这两个变量是用来让前端控制抓取程序是否进行的。

简单理解就是一个产生任务的for循环，一个消费任务的for循环，一个用于给client返回计数的for循环。这里不得不感叹，goroutine
channel的设计使得编码简单明了。

遇到的问题
----------

由于第一次正经使用Go，还是遇到不少问题的。不过需求比较简单，所以没有接触什么深入的内容。主要集中在强类型带来的问题。

### DB查询

之前写过一篇关于[database/sql](http://segmentfault.com/a/1190000003036452)的文章，这次直接用了`sqlx`这个库，可以少写不少代码，也少犯错误。但是毕竟不如laravel那么方便，所幸需要写的sql不多，临时写几个方法就搞定。同时思考，如何实现一个eloquent的api。貌似有难度。

### Json处理

强类型决定了Json的处理是个痛。之前写过一个天气预报的小程序，用的是`map[string]*json.RawMessage`
这种映射结构，然后一层一层解开json。当时没发现这是有问题的，因为如果用`RawMessage`,
字符串的引号`"`也会被保留，使得字符串结果前后多了引号。\
这次再次google了一次，发现还是得用`map[string]interface{}`来映射，然后再用`type assertion`来一层层的解开json。这是一个痛苦的过程，想起php中的`json_decode()`不禁泪流满面。

### Stop Goroutine

如何中断一个goroutine是一个问题，因为需要控制开始停止。谷歌一下很快就有结果。

        go func() {
            for {
                select {
                case <-Quit:
                    IsPreparingJobs = false
                    return
                default:
                    // to do something
                }

            }
        }()

这里设置一个`IsPreparingJobs`是用于中断后再次开始这个循环。

### Testing

Golang提供的测试工具非常方便，`go test`就能进行所有测试。从martini源码中复制了两个常用方法出来。

    func expect(t *testing.T, a interface{}, b interface{}) {
        if a != b {
            t.Errorf("Expected %v (type %v) - Got %v (type %v)", b, reflect.TypeOf(b), a, reflect.TypeOf(a))
        }
    }

    func refute(t *testing.T, a interface{}, b interface{}) {
        if a == b {
            t.Errorf("Did not expect %v (type %v) - Got %v (type %v)", b, reflect.TypeOf(b), a, reflect.TypeOf(a))
        }
    }

总结
----

感觉golang作为web开发工具，在数据格式处理方面，没有弱类型语言方便。这点node倒是非常好，json转object非常方便。也许配合Promise，node会比较好用吧。golang也有优势，goroutine非常好用，官方的库功能非常全，打包为二进制可执行文件使得部署异常容易，强类型语言效率比较高。

最后有感于前几天的shadowsockets事件，作为一个ss使用者，除了感谢无私的开发者，剩下的就只是愤怒和失望。昨天又看了老罗的T1发布会，`Born to be proud`,
`天生骄傲`，在坚持做人原则方面，老罗一直是我的楷模。期待今晚的锤子发布会。

