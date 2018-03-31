
---
date: 2016-12-31T11:34:58+08:00
title: "Goji-基于Go语言的微型web框架"
description: ""
disqus_identifier: 1485833698148125911
slug: "Goji---ji-yu--Go-yu-yan-de-wei-xing--web-kuang-jia"
source: "https://segmentfault.com/a/1190000000479653"
tags: 
- 开源项目介绍 
- 网站 
- 微框架 
- golang 
categories:
- 编程语言与开发
---

[Goji](https://goji.io/) 是一个基于 Go 的微型 web 框架，其设计受到了
Sinatra 的启发。

示例
----

一个简单的 `Hello World` 示例

    package main

    import (
            "fmt"
            "net/http"

            "github.com/zenazn/goji"
            "github.com/zenazn/goji/web"
    )

    func hello(c web.C, w http.ResponseWriter, r *http.Request) {
            fmt.Fprintf(w, "Hello, %s!", c.URLParams["name"])
    }

    func main() {
            goji.Get("/hello/:name", hello)
            goji.Serve()
    }

Goji 的代码的 `example`
目录下包含了一个[示例应用](https://github.com/zenazn/goji/tree/master/example)

特性
----

-   兼容 `net/http`
-   URL 模式（同时支持 Sinatra 风格的 `/foo/:bar` 和 正则表达式）
-   可再配置的中间件栈
-   自动支持 [Einhorn](https://github.com/stripe/einhorn)、 systemd
-   渐进停止，配合 Einhorn 可实现零下线时间的渐进重载
-   Ruby on Rails / jQuery 风格的参数解析

理念
----

-   简单。 Sinatra 风格，而不是 Rails 风格。没有魔法。
-   可组合的。可与 `net/http` 组合，可以作为 `http.Handler` 使用。
-   自由。作者很喜欢 [pat](https://github.com/bmizerany/pat)。pat
    以优雅的方式实现了 Sinatra
    风格的路由，但是它不允许程序员在处理请求的过程中使用自己的线程状态。而
    Goji 则实现了任意上下文对象。

主页
----

[goji.io](https://goji.io/)

