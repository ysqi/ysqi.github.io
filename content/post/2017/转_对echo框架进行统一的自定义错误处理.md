
---
date: 2017-05-24T09:17:32+08:00
title: "对echo框架进行统一的自定义错误处理"
description: ""
disqus_identifier: 1495588652208975503
slug: "dui--echo-kuang-jia-jin-hang-tong-yi-de-zi-ding-yi-cuo-wu-chu-li"
source: "https://segmentfault.com/a/1190000009025685"
tags: 
- golang 
- 微服务 
- api设计 
- web 
categories:
- 编程语言与开发
---

借助移动端的增长，如今 RESTful 风格的 API 已经十分流行，\
用各种语言去写后端 API 都有很成熟方便的方案，用 golang 写后端 API
更是生产力的代表，\
你可以用不输 python/ruby
这类动态语言的速度，写出性能高出一两个数量级的后端 API 。

ECHO 框架
---------

由于 golang
的标准库在网络方面已经很完善，导致框架发挥余地不大。很多高手都说，\
用什么框架，用标准库就写好了，框架只是语法糖而已，还会限制项目的发展。 \
不过我们并不是高手，语法糖也是糖，用一个趁手的框架还是能提高不少效率的。
\
要是在半年前，你让我推荐框架，我会说有很多，都各有优缺点，除了 beego
随便选一个就可以。\
但是来到2017年，一个叫 `Echo` 的框架脱颖而出。这是我目前最推荐的框架。 \
Echo 的宣传语用的是 “高性能，易扩展，极简 Go Web 框架”
。它的一些特性如下图所示：

这些特性里，HTTP/2，Auto HTTPS，听着很熟？这是我之前介绍的 Caddy
也有的特性，\
因为 golang 实现这些太容易了。还有 Middleware 里的一大堆功能也差不多。\
我们在做微服务的时候，这些通用的东西由 API Gateway 统一实现就好了，\
如果你写的是个小的独立应用的后端，这些开箱即用的功能倒是能提供很大的帮助。

其实今天我主要想说说最后一个特性里提到的，“中心化的 HTTP 错误处理”。

RESTful API 错误返回
--------------------

一个团队应当有一份 RESTful API
的规范，而在规范中应该规范响应格式，包括所有错误响应的格式。\
比如[微软的规范](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)，\
[jsonapi.org 推荐规范](http://jsonapi.org/format/upcoming/)等等。 \
大部分时候我们不需要实现的那么繁琐，我们规定一个简单的结构：

    STATUS 400 Bad Request
    {
      "error": "InvalidID",
      "message": "invalid id in your url query parameters"
    }

传统的错误响应可能只有一个伴随 HTTP Status code 的 string 类型的
message，\
如今我们把正常的响应格式变成了 JSON ，那么把错误返回也用 JSON 吧。\
除了用 JSON 之外，我们又增加了一个 error 字段，\
这个字段是一个比 Status code 要详细一个级别的 Key，\
消费端可以用这个约定的 Key 做更为灵活的错误处理。

好了，我们就用这个简单的例子进行下去，今天主题讲的是 Echo
去统一处理的方法。

Echo 怎么统一处理错误？
-----------------------

其实 Echo 的文档虽然很漂亮，但是不够详细，深入一点的内容和例子并没有。\
但一个漂亮的 golang 项目，代码即是文档，我们应该有去 godoc.org
查文档的习惯。\
我们找到 Echo 的 [GoDoc](https://godoc.org/github.com/labstack/echo)，\
看 Echo 类型：

    type Echo struct {
        Server           *http.Server
        TLSServer        *http.Server
        Listener         net.Listener
        TLSListener      net.Listener
        DisableHTTP2     bool
        Debug            bool
        HTTPErrorHandler HTTPErrorHandler
        Binder           Binder
        Validator        Validator
        Renderer         Renderer
        AutoTLSManager   autocert.Manager
        Mutex            sync.RWMutex
        Logger           Logger
        // contains filtered or unexported fields
    }

果然可以定义 **HTTPErrorHandler**, 顺着找过去，

    // HTTPErrorHandler is a centralized HTTP error handler.
    type HTTPErrorHandler func(error, Context)

它是一个传入 error 和 Context 并且没有返回值的函数。\
可是知道这些还是有点晕？并不知道怎么写这个函数啊。\
没关系，我这篇文章就是讲怎么写这个函数的。往下看吧。

定义错误结构
------------

由于 golang 是静态类型，我们干啥都需要先定义个结构，代码如下：

    type httpError struct {
        code    int
        Key     string `json:"error"`
        Message string `json:"message"`
    }

    func newHTTPError(code int, key string, msg string) *httpError {
        return &httpError{
            code:    code,
            Key:     key,
            Message: msg,
        }
    }

    // Error makes it compatible with `error` interface.
    func (e *httpError) Error() string {
        return e.Key + ": " + e.Message
    }

这里我们做了三件事

1.  定义了错误的结构，其中包含 code，key 和 message，key 和 message
    可以被导出为 JSON。

2.  做了个新建错误结构的函数，这样就可以用一行代码去新建一个错误了。

3.  给这个结构增加了 `Error` 函数，这样这个结构就成了一个 golang 的
    error 接口。

处理错误
--------

我们终于可以写上文提到的自定义函数了，先看示例代码我再做解释，然后你就能写自己的了：

    package main

    import (
        "net/http"

        "github.com/labstack/echo"
    )

    // httpErrorHandler customize echo's HTTP error handler.
    func httpErrorHandler(err error, c echo.Context) {
        var (
            code = http.StatusInternalServerError
            key  = "ServerError"
            msg  string
        )

        if he, ok := err.(*httpError); ok {
            code = he.code
            key = he.Key
            msg = he.Message
        } else if ee, ok := err.(*echo.HTTPError); ok {
            code = ee.Code
            key = http.StatusText(code)
            msg = key
        } else if config.Debug {
            msg = err.Error()
        } else {
            msg = http.StatusText(code)
        }

        if !c.Response().Committed {
            if c.Request().Method == echo.HEAD {
                err := c.NoContent(code)
                if err != nil {
                    c.Logger().Error(err)
                }
            } else {
                err := c.JSON(code, newHTTPError(code, key, msg))
                if err != nil {
                    c.Logger().Error(err)
                }
            }
        }
    }

这个函数的功能就是根据传进来的 error 和上下文 Context，组装出合适的 HTTP
响应。\
可因为 golang 的 error
是一个接口，也就是第一个参数可能传进来任何奇怪的东西，\
我们需要细心的处理一下。

第一部分我们定义了默认值作为最坏的情况，在 HTTP API
里，消费端要是看到这种最坏的情况，\
说明你要被扣奖金了，除非你可以甩锅给你依赖的模块或基础设施。

第二部分我们先看看传进来的错误是不是我们之前定义的，如果是那就太好了。如果不是的话，\
有可能是 Echo 返回的错误，比如路由或者方法没有找到之类的。如果还不是，\
看来是一个其他的未知错误，如果 Debug
开着，那还好，不用扣奖金，我们把错误明细直接返回\
到 msg 里方便调试。如果也没开 Debug ... 那只好硬着头皮返回 500
并什么信息都不给了。

第三部分你可以基本照抄，是检查上下文中是否声明这个响应已经提交了，只有没提交的时候，\
我们才需要把我们准备好的错误信息以 JSON
格式提交，顺便打印错误日志。另外，如果请求\
是 HEAD 方法的话，根据规范，你只能返回状态 204 并默默在日志记录错误了。

应用
----

好了，我们写好了统一的错误处理，该怎么使用呢？ 来看一个极简的例子吧：

    func getUser(c echo.Context) error {
        var u user
        id := c.Param("id")
        if !bson.IsObjectIdHex(id) {
            return newHTTPError(http.StatusBadRequest, "InvalidID", "invalid user id")
        }
        err := db.C("user").FindId(bson.ObjectIdHex(id)).One(&u)
        if err == mgo.ErrNotFound {
            return newHTTPError(http.StatusNotFound, "NotFound", err.Error())
        }
        if err != nil {
            return err
        }
        return c.JSON(http.StatusOK, u)
    }

这是个从 mongodb 取 user 的例子，

1.  检查url中的id是不是一个合法的id，不是的话，返回我们之前自定义的错误。

2.  去数据库里查，如果没有记录，返回 404 错误。

3.  如果查询数据库的操作出了其他错误，这个时候我们无能为力了，只好直接把这个错误返回。

4.  一切正常没错误的话，我们返回状态 200 和 JSON 数据。

我们可以看出，经过这么一番折腾，在写API的时候，省心了很多。\
我们可以随手用一行代码构造错误，也可以直接把任何预测不到的错误返回，\
不用再麻烦的每次去构造 500 错误了。

怎么样？快去安利小伙伴们用 echo 写 HTTP API 吧，真的很方便。

