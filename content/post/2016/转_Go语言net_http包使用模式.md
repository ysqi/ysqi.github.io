
---
date: 2016-12-31T11:33:08+08:00
title: "Go语言net_http包使用模式"
description: ""
disqus_identifier: 1485833588174874478
slug: "Go-yu-yan-net_http-bao-shi-yong-mo-shi"
source: "https://segmentfault.com/a/1190000006812688"
tags: 
- go-web-framework 
- golang 
categories:
- 编程语言与开发
---

译注:
这篇文章的内容非常基础，也非常容易理解。[原文地址](http://www.alexedwards.net/blog/a-recap-of-request-handling)，感觉是最能清晰的讲述了`net/http`包的用法的一篇，故翻译一下共享之。

一切的基础：ServeMux 和 Handler
-------------------------------

Go 语言中处理 HTTP 请求主要跟两个东西相关：`ServeMux` 和 `Handler`。

[`ServrMux`](http://golang.org/pkg/net/http/#ServeMux) 本质上是一个 HTTP
请求路由器（或者叫多路复用器，Multiplexor）。它把收到的请求与一组预先定义的
URL 路径列表做对比，然后在匹配到路径的时候调用关联的处理器（Handler）。

**处理器（Handler）**负责输出HTTP响应的头和正文。任何满足了[`http.Handler`接口](http://golang.org/pkg/net/http/#Handler)的对象都可作为一个处理器。通俗的说，对象只要有个如下签名的`ServeHTTP`方法即可：

    ServeHTTP(http.ResponseWriter, *http.Request)

Go 语言的 HTTP
包自带了几个函数用作常用处理器，比如[`FileServer`](http://golang.org/pkg/net/http/#FileServer)，[`NotFoundHandler`](http://golang.org/pkg/net/http/#NotFoundHandler)
和
[`RedirectHandler`](http://golang.org/pkg/net/http/#RedirectHandler)。我们从一个简单具体的例子开始：

    $ mkdir handler-example
    $ cd handler-example
    $ touch main.go

    //File: main.go
    package main

    import (
      "log"
      "net/http"
    )

    func main() {
      mux := http.NewServeMux()

      rh := http.RedirectHandler("http://example.org", 307)
      mux.Handle("/foo", rh)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

快速地过一下代码：

-   在 `main` 函数中我们只用了
    [`http.NewServeMux`](http://golang.org/pkg/net/http/#NewServeMux)
    函数来创建一个空的 `ServeMux`。

-   然后我们使用
    [`http.RedirectHandler`](http://golang.org/pkg/net/http/#RedirectHandler)
    函数创建了一个新的处理器，这个处理器会对收到的所有请求，都执行307重定向操作到
    `http://example.org`。

-   接下来我们使用
    [`ServeMux.Handle`](http://golang.org/pkg/net/http/#ServeMux.Handle)
    函数将处理器注册到新创建的 `ServeMux`，所以它在 URL 路径`/foo`
    上收到所有的请求都交给这个处理器。

-   最后我们创建了一个新的服务器，并通过
    [`http.ListenAndServe`](http://golang.org/pkg/net/http/#ListenAndServe)
    函数监听所有进入的请求，通过传递刚才创建的
    `ServeMux`来为请求去匹配对应处理器。

继续，运行一下这个程序:

    $ go run main.go
    Listening...

然后在浏览器中访问
`http://localhost:3000/foo`，你应该能发现请求已经成功的重定向了。

明察秋毫的你应该能注意到一些有意思的事情：`ListenAndServer` 的函数签名是
`ListenAndServe(addr string, handler Handler)`
，但是第二个参数我们传递的是个 `ServeMux`。

我们之所以能这么做，是因为 `ServeMux` 也有 `ServeHTTP`
方法，因此它也是个合法的 `Handler`。

对我来说，将 `ServerMux`
用作一个特殊的Handler是一种简化。它不是自己输出响应而是将请求传递给注册到它的其他
Handler。这乍一听起来不是什么明显的飞跃 - 但在 Go 中将 `Handler`
链在一起是非常普遍的用法。

### 自定义处理器（Custom Handlers）

让我们创建一个自定义的处理器，功能是将以特定格式输出当前的本地时间：

    type timeHandler struct {
      format string
    }

    func (th *timeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
      tm := time.Now().Format(th.format)
      w.Write([]byte("The time is: " + tm))
    }

这个例子里代码本身并不是重点。

真正的重点是我们有一个对象（本例中就是个`timerHandler`结构体，但是也可以是一个字符串、一个函数或者任意的东西），我们在这个对象上实现了一个
`ServeHTTP(http.ResponseWriter, *http.Request)`
签名的方法，这就是我们创建一个处理器所需的全部东西。

我们把这个集成到具体的示例里：

    //File: main.go

    package main

    import (
      "log"
      "net/http"
      "time"
    )

    type timeHandler struct {
      format string
    }

    func (th *timeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
      tm := time.Now().Format(th.format)
      w.Write([]byte("The time is: " + tm))
    }

    func main() {
      mux := http.NewServeMux()

      th := &timeHandler{format: time.RFC1123}
      mux.Handle("/time", th)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

`main`函数中，我们像初始化一个常规的结构体一样，初始化了`timeHandler`，用
`&` 符号获得了其地址。随后，像之前的例子一样，我们使用 `mux.Handle`
函数来将其注册到 `ServerMux`。

现在当我们运行这个应用，`ServerMux` 将会将任何对 `/time`的请求直接交给
`timeHandler.ServeHTTP` 方法处理。

访问一下这个地址看一下效果：<http://localhost:3000/time> 。

注意我们可以在多个路由中轻松的复用 `timeHandler`：

    func main() {
      mux := http.NewServeMux()

      th1123 := &timeHandler{format: time.RFC1123}
      mux.Handle("/time/rfc1123", th1123)

      th3339 := &timeHandler{format: time.RFC3339}
      mux.Handle("/time/rfc3339", th3339)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

### 将函数作为处理器

对于简单的情况（比如上面的例子），定义个新的有 `ServerHTTP`
方法的自定义类型有些累赘。我们看一下另外一种方式，我们借助
[`http.HandlerFunc`](http://golang.org/pkg/net/http/#HandlerFunc)
类型来让一个常规函数满足作为一个 `Handler` 接口的条件。

任何有 `func(http.ResponseWriter, *http.Request)`
签名的函数都能转化为一个 `HandlerFunc` 类型。这很有用，因为
`HandlerFunc` 对象内置了 `ServeHTTP`
方法，后者可以聪明又方便的调用我们最初提供的函数内容。

如果你听起来还有些困惑，可以尝试看一下\[相关的源代码\][http://golang.org/src/pkg/net...](http://golang.org/src/pkg/net/http/server.go?s=35455:35502#L1221())。你将会看到让一个函数对象满足
`Handler` 接口是非常简洁优雅的。

让我们使用这个技术重新实现一遍`timeHandler`应用：

    //File: main.go
    package main

    import (
      "log"
      "net/http"
      "time"
    )

    func timeHandler(w http.ResponseWriter, r *http.Request) {
      tm := time.Now().Format(time.RFC1123)
      w.Write([]byte("The time is: " + tm))
    }

    func main() {
      mux := http.NewServeMux()

      // Convert the timeHandler function to a HandlerFunc type
      th := http.HandlerFunc(timeHandler)
      // And add it to the ServeMux
      mux.Handle("/time", th)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

实际上，将一个函数转换成 `HandlerFunc` 后注册到 `ServeMux`
是很普遍的用法，所以 Go
语言为此提供了个便捷方式：[`ServerMux.HandlerFunc`](http://golang.org/pkg/net/http/#ServeMux.HandleFunc)
方法。

我们使用便捷方式重写 `main()` 函数看起来是这样的：

    func main() {
      mux := http.NewServeMux()

      mux.HandleFunc("/time", timeHandler)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

绝大多数情况下这种用函数当处理器的方式工作的很好。但是当事情开始变得更复杂的时候，就会有些产生一些限制了。

你可能已经注意到了，跟之前的方式不同，我们不得不将时间格式硬编码到
`timeHandler` 的方法中。如果我们想从 `main()`
函数中传递一些信息或者变量给处理器该怎么办？

一个优雅的方式是将我们处理器放到一个闭包中，将我们要使用的变量带进去：

    //File: main.go
    package main

    import (
      "log"
      "net/http"
      "time"
    )

    func timeHandler(format string) http.Handler {
      fn := func(w http.ResponseWriter, r *http.Request) {
        tm := time.Now().Format(format)
        w.Write([]byte("The time is: " + tm))
      }
      return http.HandlerFunc(fn)
    }

    func main() {
      mux := http.NewServeMux()

      th := timeHandler(time.RFC1123)
      mux.Handle("/time", th)

      log.Println("Listening...")
      http.ListenAndServe(":3000", mux)
    }

`timeHandler` 函数现在有了个更巧妙的身份。除了把一个函数封装成
Handler(像我们之前做到那样)，我们现在使用它来返回一个处理器。这种机制有两个关键点：

首先是创建了一个`fn`，这是个匿名函数，将 `format`
变量封装到一个闭包里。闭包的本质让处理器在任何情况下，都可以在本地范围内访问到
`format` 变量。

其次我们的闭包函数满足 `func(http.ResponseWriter, *http.Request)`
签名。如果你记得之前我们说的，这意味我们可以将它转换成一个`HandlerFunc`类型（满足了`http.Handler`接口）。我们的`timeHandler`
函数随后转换后的 `HandlerFunc` 返回。

在上面的例子中我们已经可以传递一个简单的字符串给处理器。但是在实际的应用中可以使用这种方法传递数据库连接、模板组，或者其他应用级的上下文。使用全局变量也是个不错的选择，还能得到额外的好处就是编写更优雅的自包含的处理器以便测试。

你也可能见过相同的写法，像这样：

    func timeHandler(format string) http.Handler {
      return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        tm := time.Now().Format(format)
        w.Write([]byte("The time is: " + tm))
      })
    }

或者在返回时，使用一个到 `HandlerFunc` 类型的隐式转换：

    func timeHandler(format string) http.HandlerFunc {
      return func(w http.ResponseWriter, r *http.Request) {
        tm := time.Now().Format(format)
        w.Write([]byte("The time is: " + tm))
      }
    }

### 更便利的 DefaultServeMux

你可能已经在很多地方看到过 `DefaultServeMux`, 从最简单的 `Hello World`
例子，到 go 语言的源代码中。

我花了很长时间才意识到 `DefaultServerMux`
并没有什么的特殊的地方。`DefaultServerMux` 就是我们之前用到的
`ServerMux`，只是它随着 `net/httpp` 包初始化的时候被自动初始化了而已。Go
源代码中的相关行如下：

    var DefaultServeMux = NewServeMux()

`net/http` 包提供了一组快捷方式来配合
`DefaultServeMux`：[`http.Handle`](http://golang.org/pkg/net/http/#Handle)
和
[`http.HandleFunc`](http://golang.org/pkg/net/http/#HandleFunc)。这些函数与我们之前看过的类似的名称的函数功能一样，唯一的不同是他们将处理器注册到
`DefaultServerMux` ，而之前我们是注册到自己创建的 `ServeMux`。

此外，`ListenAndServe`在没有提供其他的处理器的情况下（也就是第二个参数设成了
`nil`），内部会使用 `DefaultServeMux`。

因此，作为最后一个步骤，我们使用 `DefaultServeMux` 来改写我们的
`timeHandler`应用：

    //File: main.go
    package main

    import (
      "log"
      "net/http"
      "time"
    )

    func timeHandler(format string) http.Handler {
      fn := func(w http.ResponseWriter, r *http.Request) {
        tm := time.Now().Format(format)
        w.Write([]byte("The time is: " + tm))
      }
      return http.HandlerFunc(fn)
    }

    func main() {
      // Note that we skip creating the ServeMux...

      var format string = time.RFC1123
      th := timeHandler(format)

      // We use http.Handle instead of mux.Handle...
      http.Handle("/time", th)

      log.Println("Listening...")
      // And pass nil as the handler to ListenAndServe.
      http.ListenAndServe(":3000", nil)
    }

