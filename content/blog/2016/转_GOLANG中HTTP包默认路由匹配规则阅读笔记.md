
---
date: 2016-12-31T11:33:31+08:00
title: "GOLANG中HTTP包默认路由匹配规则阅读笔记"
description: ""
disqus_identifier: 1485833611618361508
slug: "GOLANG-zhong-HTTPbao-mo-ren-lu-you-pi-pei-gui-ze-yue-dou-bi-ji"
source: "https://segmentfault.com/a/1190000005591382"
tags: 
- golang 
topics:
- 编程语言与开发
---

一、执行流程
============

构建一个简单http server：

    package main

    import (
        "log"
        "net/http"
    )

    func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            w.Write([]byte("hello world"))
        })
        log.Fatal(http.ListenAndServe(":8080", nil))
    }

使用`http://127.0.0.1:8080/` 就可以看到输出了

通过跟踪http.go包代码，可以发现执行流程基本如下：

#### 1.创建一个`Listener`监听`8080`端口

#### 2.进入`for`循环并Accept请求，没有请求则处于阻塞状态

#### 3.接收到请求，并创建一个conn对象，放入goroutine处理（实现高并发关键）

#### 4.解析请求来源信息获得请求路径等重要信息

#### 5.请求ServerHTTP方法，已经通过上一步获得了ResponseWriter和Request对象

    func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
        //此handler即为http.ListenAndServe 中的第二个参数
        handler := sh.srv.Handler 
        if handler == nil {
            //如果handler为空则使用内部的DefaultServeMux 进行处理
            handler = DefaultServeMux
        }
        if req.RequestURI == "*" && req.Method == "OPTIONS" {
            handler = globalOptionsHandler{}
        }
        //这里就开始处理http请求
        //如果需要使用自定义的mux，就需要实现ServeHTTP方法，即实现Handler接口。
        handler.ServeHTTP(rw, req)
    }

6.进入DefaultServeMux中的逻辑就是根据请求path在map中匹配查找handler，并交由handler处理

> http请求处理流程更多信息可以参考\[《Go Web 编程\
> 》3.3
> Go如何使得Web工作\](<https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/03.3.md)>

二、DefaultServeMux 路由匹配规则
================================

先看几个路由规则：

    package main

    import (
        "log"
        "net/http"
    )

    func main() {
        //规则1
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            w.Write([]byte("hello world"))
        })
        
        //规则2
        http.HandleFunc("/path/", func(w http.ResponseWriter, r *http.Request) {
            w.Write([]byte("pattern path: /path/ "))
        })

        //规则3
        http.HandleFunc("/path/subpath", func(w http.ResponseWriter, r *http.Request) {
            w.Write([]byte("pattern path: /path/subpath"))
        })

        log.Fatal(http.ListenAndServe(":8080", nil))
    }

情景一：

访问：`http://127.0.0.1:8080/`

返回：`hello world`

情景二：

访问：`http://127.0.0.1:8080/path`

返回：`pattern path: /path/ `

情景三：

访问：`http://127.0.0.1:8080/path/subpath/`

返回：`pattern path: /path/ `

情景四：

访问：`http://127.0.0.1:8080/hahaha/`

返回：`hello world`

先说明一些规则吧，再看代码是怎么实现的：

1.如果匹配路径中后带有`/`，则会自动增加一个匹配规则不带`/`后缀的，并跳转转到`path/`,解释了情景二的场景，为什么匹配到的`/path/`

2.我设置了这么多规则为什么规则一可以通用匹配未设置的路由信息，而且又不影响已经存在路由，
内部是怎么实现的？

2.1 添加路由规则
----------------

先看两个struct，这是存放默认路由规则的：

    type ServeMux struct {
        mu    sync.RWMutex  //处理并发，增加读写锁
        m     map[string]muxEntry  //存放规则map，key即为设置的path
        hosts bool // whether any patterns contain hostnames（是否包含host）
    }

    type muxEntry struct {
        explicit bool //是否完全匹配
        h        Handler//相应匹配规则的handler
        pattern  string//匹配路径
    }

通过跟踪`http.HandleFunc`定位到如下代码，正是往上面两个`struct`中增加规则：

    func (mux *ServeMux) Handle(pattern string, handler Handler) {
        mux.mu.Lock()
        defer mux.mu.Unlock()

        if pattern == "" {
            panic("http: invalid pattern " + pattern)
        }
        if handler == nil {
            panic("http: nil handler")
        }
        //如果已经匹配到了则panic
        if mux.m[pattern].explicit {
            panic("http: multiple registrations for " + pattern)
        }
        
        //增加一个新的匹配规则
        mux.m[pattern] = muxEntry{explicit: true, h: handler, pattern: pattern}
        
        //根据path的第一个字母判断是否有host
        if pattern[0] != '/' {
            mux.hosts = true
        }

        //！！这里看清楚 就是实现了情景二的情况 ，看判断条件
        n := len(pattern)
        if n > 0 && pattern[n-1] == '/' && !mux.m[pattern[0:n-1]].explicit{
            // If pattern contains a host name, strip it and use remaining
            // path for redirect.
            path := pattern
            if pattern[0] != '/' {
                // In pattern, at least the last character is a '/', so
                // strings.Index can't be -1.
                path = pattern[strings.Index(pattern, "/"):]
            }
            url := &url.URL{Path: path}
            mux.m[pattern[0:n-1]] = muxEntry{h: RedirectHandler(url.String(), StatusMovedPermanently), pattern: pattern}
        }
    }

上面有个`Helpful behavior`的注释行为，就是实现了情景二的情况，他是判断如果匹配的路径中最后含有`/`，并且之前也不存在添加去除反斜杠的规则的话，就自动给他增加一个301的跳转指向`/path/`

2.2 查找路由规则
----------------

路由规则的查找就是从`ServeMux `中的map去匹配查找的,的到这个handler并执行，只是会有一些处理机制，比如怎么样确保访问`/path/subpath`的时候是先匹配`/path/subpath`而不是匹配`/path/`呢？

当一个请求过来的时候，跟踪到了`mux.match`方法：

> 过程`mux.ServerHTTP`-&gt;`mux.Handler`-&gt;`mux.handler`-&gt;`mux.match`

    func (mux *ServeMux) match(path string) (h Handler, pattern string) {
        var n = 0
        for k, v := range mux.m {
            if !pathMatch(k, path) {
                continue
            }
            //如果匹配到了一个规则，并没有马上返回handler，而且继续匹配并且判断path的长度是否是最长的，这是关键！！！
            if h == nil || len(k) > n {
                n = len(k)
                h = v.h
                pattern = v.pattern
            }
        }
        return
    }

1.这里就解释了为什么设置的精确的path是最优匹配到的，因为它是根据path的长度判断。\
当然也就解释了为什么`/`可以匹配所有（看`pathMatch`函数就知道了，`/`是匹配所有的，只是这是最后才被匹配成功）

2.得到了处理请求的handler，再调用`h.ServeHTTP(w, r)`，去执行相应的handler方法。

等一下，handler中哪里有`ServeHTTP`这个方法？？

因为在调用
`http.HandleFunc`的时候已经将自定义的handler处理函数，强制转为`HandlerFunc`类型的，就拥有了`ServeHTTP`方法：

    type HandlerFunc func(ResponseWriter, *Request)

    // ServeHTTP calls f(w, r).
    func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
        f(w, r)
    }

`f(w,r)`就实现了handler的执行。

原文地址：[silenceper.com](http://silenceper.com/blog/201605/golang-%E4%B8%ADhttp%E5%8C%85%E9%BB%98%E8%AE%A4%E8%B7%AF%E7%94%B1%E5%8C%B9%E9%85%8D%E8%A7%84%E5%88%99%E9%98%85%E8%AF%BB%E7%AC%94%E8%AE%B0/)

