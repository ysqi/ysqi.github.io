
---
date: 2016-12-31T11:33:10+08:00
title: "Go语言并发模型:使用context"
description: ""
disqus_identifier: 1485833590631189474
slug: "Goyu-yan-bing-fa-mo-xing-:shi-yong--context"
source: "https://segmentfault.com/a/1190000006744213"
tags: 
- context 
- golang 
categories:
- 编程语言与开发
---

简介
----

在 Go http包的Server中，每一个请求在都有一个对应的 goroutine
去处理。请求处理函数通常会启动额外的 goroutine
用来访问后端服务，比如数据库和RPC服务。用来处理一个请求的 goroutine
通常需要访问一些与请求特定的数据，比如终端用户的身份认证信息、验证相关的token、请求的截止时间。
当一个请求被取消或超时时，所有用来处理该请求的 goroutine
都应该迅速退出，然后系统才能释放这些 goroutine 占用的资源。

在Google 内部，我们开发了 `Context` 包，专门用来简化
对于处理单个请求的多个 goroutine
之间与请求域的数据、取消信号、截止时间等相关操作，这些操作可能涉及多个
API 调用。你可以通过 `go get golang.org/x/net/context`
命令获取这个包。本文要讲的就是如果使用这个包，同时也会提供一个完整的例子。

阅读建议
--------

本文内容涉及到了 done channel，如果你不了解这个概念，那么请先阅读
["Go语言并发模型：像Unix
Pipe那样使用channel"](https://segmentfault.com/a/1190000006261218)。

由于访问 `golang.org/x/net/context` 需要梯子，你可以访问它在 github 上的
[mirror](https://github.com/golang/net)。\
如果要下载本文中的代码，可以查看文章末尾的“相关链接”环节。

package context
---------------

context 包的核心是 struct Context，声明如下：

    // A Context carries a deadline, cancelation signal, and request-scoped values
    // across API boundaries. Its methods are safe for simultaneous use by multiple
    // goroutines.
    type Context interface {
        // Done returns a channel that is closed when this `Context` is canceled
        // or times out.
        Done() <-chan struct{}

        // Err indicates why this Context was canceled, after the Done channel
        // is closed.
        Err() error

        // Deadline returns the time when this Context will be canceled, if any.
        Deadline() (deadline time.Time, ok bool)

        // Value returns the value associated with key or nil if none.
        Value(key interface{}) interface{}
    }

注意: 这里我们对描述进行了简化，更详细的描述查看
[godoc:context](http://godoc.org/golang.org/x/net/context)

`Done` 方法返回一个 channel，这个 channel 对于以 `Context`
方式运行的函数而言，是一个取消信号。当这个 channel
关闭时，上面提到的这些函数应该终止手头的工作并立即返回。 之后，`Err`
方法会返回一个错误，告知为什么 `Context` 被取消。关于 `Done` channel
的更多细节查看上一篇文章 ["Go语言并发模型：像Unix
Pipe那样使用channel"](https://segmentfault.com/a/1190000006261218)。

一个 `Context` 不能拥有 `Cancel` 方法，同时我们也只能 `Done` channel
接收数据。背后的原因是一致的：接收取消信号的函数和发送信号的函数通常不是一个。
一个典型的场景是：父操作为子操作操作启动
goroutine，子操作也就不能取消父操作。 作为一个折中，`WithCancel` 函数
(后面会细说) 提供了一种取消新的 `Context` 的方法。

`Context` 对象是线程安全的，你可以把一个 `Context` 对象传递给任意个数的
gorotuine，\
对它执行 取消 操作时，所有 goroutine 都会接收到取消信号。

`Deadline`
方法允许函数确定它们是否应该开始工作。如果剩下的时间太少，也许这些函数就不值得启动。代码中，我们也可以使用
`Deadline` 对象为 I/O 操作设置截止时间。

`Value` 方法允许 `Context`
对象携带request作用域的数据，该数据必须是线程安全的。

继承 context
------------

context 包提供了一些函数，协助用户从现有的 `Context` 对象创建新的
`Context` 对象。\
这些 `Context` 对象形成一棵树：当一个 `Context`
对象被取消时，继承自它的所有 `Context` 都会被取消。

`Background` 是所有 `Context` 对象树的根，它不能被取消。它的声明如下：

    // Background returns an empty Context. It is never canceled, has no deadline,
    // and has no values. Background is typically used in main, init, and tests,
    // and as the top-level `Context` for incoming requests.
    func Background() Context

`WithCancel` 和 `WithTimeout` 函数 会返回继承的 `Context` 对象，
这些对象可以比它们的父 `Context` 更早地取消。

当请求处理函数返回时，与该请求关联的 `Context` 会被取消。
当使用多个副本发送请求时，可以使用 `WithCancel`取消多余的请求。
`WithTimeout` 在设置对后端服务器请求截止时间时非常有用。
下面是这三个函数的声明：

    // WithCancel returns a copy of parent whose Done channel is closed as soon as
    // parent.Done is closed or cancel is called.
    func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

    // A CancelFunc cancels a Context.
    type CancelFunc func()

    // WithTimeout returns a copy of parent whose Done channel is closed as soon as
    // parent.Done is closed, cancel is called, or timeout elapses. The new
    // Context's Deadline is the sooner of now+timeout and the parent's deadline, if
    // any. If the timer is still running, the cancel function releases its
    // resources.
    func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)

`WithValue` 函数能够将请求作用域的数据与 `Context`
对象建立关系。声明如下：

    // WithValue returns a copy of parent whose Value method returns val for key.
    func WithValue(parent Context, key interface{}, val interface{}) Context

当然，想要知道 `Context` 包是如何工作的，最好的方法是看一个栗子。

一个栗子：Google Web Search
---------------------------

我们的例子是一个 HTTP 服务，它能够将类似于 `/search?q=golang&timeout=1s`
的请求 转发给\
[Google Web Search
API](https://developers.google.com/web-search/docs/)，然后渲染返回的结果。`timeout`
参数用来告诉 server 时间到时取消请求。

这个例子的代码存放在三个包里：

1.  [server](https://blog.golang.org/context/server/server.go)：它提供
    main 函数和 处理 `/search` 的 http handler

2.  [userip](https://blog.golang.org/context/userip/userip.go)：它能够从
    请求解析用户的IP，并将请求绑定到一个 `Context` 对象。

3.  [google](https://blog.golang.org/context/google/google.go)：它包含了
    Search 函数，用来向 Google 发送请求。

### 深入 server 程序

[server](https://blog.golang.org/context/server/server.go)
程序处理类似于 `/search?q=golang` 的请求，返回 Google API
的搜索结果。它将 `handleSearch` 函数注册到 `/search`
路由。处理函数创建一个 `Context` ctx，并对其进行初始化，以保证 `Context`
取消时，处理函数返回。如果请求的 URL 参数中包含 `timeout`，那么当
`timeout` 到期时， `Context` 会被自动取消。\
handleSearch 的代码如下：

    func handleSearch(w http.ResponseWriter, req *http.Request) {
        // ctx is the `Context` for this handler. Calling cancel closes the
        // ctx.Done channel, which is the cancellation signal for requests
        // started by this handler.
        var (
            ctx    context.Context
            cancel context.CancelFunc
        )
        timeout, err := time.ParseDuration(req.FormValue("timeout"))
        if err == nil {
            // The request has a timeout, so create a `Context` that is
            // canceled automatically when the timeout expires.
            ctx, cancel = context.WithTimeout(context.Background(), timeout)
        } else {
            ctx, cancel = context.WithCancel(context.Background())
        }
        defer cancel() // Cancel ctx as soon as handleSearch returns.

处理函数 (handleSearch) 将query 参数从请求中解析出来，然后通过 userip
包将client IP解析出来。这里 Client IP 在后端发送请求时要用到，所以
handleSearch 函数将它 attach 到 `Context` 对象 ctx 上。代码如下：

    // Check the search query.
    query := req.FormValue("q")
    if query == "" {
        http.Error(w, "no query", http.StatusBadRequest)
        return
    }

    // Store the user IP in ctx for use by code in other packages.
    userIP, err := userip.FromRequest(req)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    ctx = userip.NewContext(ctx, userIP)

处理函数带着 `Context` 对象 `ctx` 和 `query` 调用
`google.Search`，代码如下：

    // Run the Google search and print the results.
    start := time.Now()
    results, err := google.Search(ctx, query)
    elapsed := time.Since(start)

如果搜索成功，处理函数会渲染搜索结果，代码如下：

    if err := resultsTemplate.Execute(w, struct {
        Results          google.Results
        Timeout, Elapsed time.Duration
    }{
        Results: results,
        Timeout: timeout,
        Elapsed: elapsed,
    }); err != nil {
        log.Print(err)
        return
    }

### 深入 userip 包

[userip](https://blog.golang.org/context/userip/userip.go)
包提供了两个功能：

1.  从请求解析出Client IP；

2.  将 Client IP 关联到一个 `Context` 对象。

一个 `Context` 对象提供一个 key-value 映射，key 和 value的类型都是
interface{}，但是 key 必须满足等价性（可以比较），value
必须是线程安全的。类似于 `userip` 的包隐藏了映射的细节，提供的是对特定
`Context` 类型值得强类型访问。

为了避免 key 冲突，`userip` 定义了一个非输出类型
`key`，并使用该类型的值作为 `Context` 的key。代码如下：

    // 为了避免与其他包中的 `Context` key 冲突
    // 这里不输出 key 类型 (首字母小写)
    type key int

    // userIPKey 是 user IP 的 `Context` key
    // 它的值是随意写的。如果这个包中定义了其他
    // `Context` key，这些 key 必须不同
    const userIPKey key = 0

函数 `FromRequest` 用来从一个 http.Request 对象中解析出 userIP：

    func FromRequest(req *http.Request) (net.IP, error) {
        ip, _, err := net.SplitHostPort(req.RemoteAddr)
        if err != nil {
            return nil, fmt.Errorf("userip: %q is not IP:port", req.RemoteAddr)
        }

函数 `NewContext` 返回一个新的 `Context` 对象，它携带者 userIP：

    func NewContext(ctx context.Context, userIP net.IP) context.Context {
        return context.WithValue(ctx, userIPKey, userIP)
    }

函数 `FromContext` 从一个 `Context` 对象中解析 userIP：

    func FromContext(ctx context.Context) (net.IP, bool) {
        // ctx.Value returns nil if ctx has no value for the key;
        // the net.IP type assertion returns ok=false for nil.
        userIP, ok := ctx.Value(userIPKey).(net.IP)
        return userIP, ok
    }

### 深入 google 包

函数 `google.Search` 想 Google Web Search API 发送一个 HTTP
请求，并解析返回的 JSON 数据。该函数接收一个 `Context` 对象 ctx
作为第一参数，在请求还没有返回时，一旦 `ctx.Done`
关闭，该函数也会立即返回。

Google Web Search API 请求包含 query 关键字和 user IP
两个参数。具体实现如下：

    func Search(ctx context.Context, query string) (Results, error) {
        // Prepare the Google Search API request.
        req, err := http.NewRequest("GET", "https://ajax.googleapis.com/ajax/services/search/web?v=1.0", nil)
        if err != nil {
            return nil, err
        }
        q := req.URL.Query()
        q.Set("q", query)

        // If ctx is carrying the user IP address, forward it to the server.
        // Google APIs use the user IP to distinguish server-initiated requests
        // from end-user requests.
        if userIP, ok := userip.FromContext(ctx); ok {
            q.Set("userip", userIP.String())
        }
        req.URL.RawQuery = q.Encode()

函数 `Search` 使用一个辅助函数 `httpDo` 发送 HTTP 请求，并在 `ctx.Done`
关闭时取消请求 (如果还在处理请求或返回)。函数 `Search` 传递给 `httpDo`
一个闭包处理 HTTP 结果。下面是具体实现：

    var results Results
    err = httpDo(ctx, req, func(resp *http.Response, err error) error {
        if err != nil {
            return err
        }
        defer resp.Body.Close()

        // Parse the JSON search result.
        // https://developers.google.com/web-search/docs/#fonje
        var data struct {
            ResponseData struct {
                Results []struct {
                    TitleNoFormatting string
                    URL               string
                }
            }
        }
        if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
            return err
        }
        for _, res := range data.ResponseData.Results {
            results = append(results, Result{Title: res.TitleNoFormatting, URL: res.URL})
        }
        return nil
    })
    // httpDo waits for the closure we provided to return, so it's safe to
    // read results here.
    return results, err

函数 `httpDo` 在一个新的 goroutine 中发送 HTTP 请求和处理结果。如果
`ctx.Done` 已经关闭，而处理请求的 goroutine
还存在，那么取消请求。下面是具体实现：

    func httpDo(ctx context.Context, req *http.Request, f func(*http.Response, error) error) error {
        // Run the HTTP request in a goroutine and pass the response to f.
        tr := &http.Transport{}
        client := &http.Client{Transport: tr}
        c := make(chan error, 1)
        go func() { c <- f(client.Do(req)) }()
        select {
        case <-ctx.Done():
            tr.CancelRequest(req)
            <-c // Wait for f to return.
            return ctx.Err()
        case err := <-c:
            return err
        }
    }

在自己的代码中使用 `Context`
----------------------------

许多服务器框架都提供了管理请求作用域数据的包和类型。我们可以定义一个
`Context` 接口的实现，\
将已有代码和期望 `Context` 参数的代码粘合起来。

举个栗子，Gorilla 框架的
[github.com/gorilla/context](http://www.gorillatoolkit.org/pkg/context)
包允许处理函数 (handlers) 将数据和请求结合起来，他通过 HTTP 请求 到
key-value对 的映射来实现。在
[gorilla.go](https://blog.golang.org/context/gorilla/gorilla.go)
中，我们提供了一个 `Context` 的具体实现，这个实现的 Value
方法返回的值已经与 gorilla 包中特定的 HTTP 请求关联起来。

还有一些包实现了类似于 `Context` 的取消机制。比如
[Tomb](http://godoc.org/gopkg.in/tomb.v2) 中有一个 Kill
方法，该方法通过关闭 名为`Dying` 的 channel 发送取消信号。`Tomb`
也提供了等待 goroutine 退出的方法，类似于 `sync.WaitGroup`。在
[tomb.go](https://blog.golang.org/context/tomb/tomb.go)
中，我们提供了一个 `Context` 的实现，当它的父 `Context` 被取消\
或 一个 `Tomb` 对象被 kill 时，该 `Context` 对象也会被取消。

结论
----

在 Google， 我们要求 Go 程序员把 `Context` 作为第一个参数传递给
入口请求和出口请求链路上的每一个函数。这种机制一方面保证了多个团队开发的
Go
项目能够良好地协作，另一方面它是一种简单的超时和取消机制，保证了临界区数据
(比如安全凭证) 在不同的 Go 项目中顺利传递。

如果你要在 `Context` 之上构建服务器框架，需要一个自己的 `Context`
实现，在框架与期望 `Context` 参数的代码之间建立一座桥梁。\
当然，Client 库也需要接收一个 `Context`
对象。在请求作用域数据与取消之间建立了通用的接口以后，开发者使用
Context\
分享代码、创建可扩展的服务都会非常方便。

原作者：Sameer Ajmani 翻译：Oscar

下期预告：Go语言并发模型：使用 select
([原文链接](https://talks.golang.org/2012/concurrency.slide#31))。

相关链接
--------

1.  [原文链接](https://blog.golang.org/context)

2.  [代码位置](https://blog.golang.org/context/)

3.  [代码位置(mirror)](https://github.com/oscarzhao/golang/tree/master/go_blog)

4.  [mirror of package net](https://github.com/golang/net)

5.  [Google Web Search
    API](https://developers.google.com/web-search/docs/)

------------------------------------------------------------------------

扫码关注微信公众号“深入Go语言”



