
---
date: 2016-12-31T11:35:10+08:00
title: "用Go语言写HTTP中间件"
description: ""
disqus_identifier: 1485833710569337558
slug: "yong-Goyu-yan-xie-HTTPzhong-jian-jian"
source: "https://segmentfault.com/a/1190000000358436"
tags: 
- http 
- golang 
categories:
- 编程语言与开发
---

在web开发过程中，中间件一般是指应用程序中封装原始信息，添加额外功能的组件。不知道为什么，中间件通常是一种不太受欢迎的概念。但我认为它棒极了。

其一，一个好的中间件拥有单一的功能，可插拔并且是自我约束的。这就意味着你可以在接口的层次上把它放到应用中，并能很好的工作。中间件并不影响你的代码风格，它也不是一个框架，仅仅是你处理请求流程中额外一层罢了。根本不需要重写代码：如果你想用一个中间件，就把它加上应用中；如果你改变主意了，去掉就好了。就这么简单。

来看看Go，HTTP中间件非常流行，标准库中也是这样。或许咋看上去并不明显，net/http包中的函数，如[StripPrefix](http://golang.org/pkg/net/http/#StripPrefix)
和[TimeoutHandler](http://golang.org/pkg/net/http/#TimeoutHandler)
正是我们上面定义的中间件：封装处理过程并在处理输入或输出时增加额外的动作。

我最近的Go包 [nosurf](https://github.com/justinas/nosurf)
也是一个中间件。我从一开始就有意的这样设计。大多数情况下，你根本不必在应用层关心CSRF检查。nosurf，和其他中间件一样，非常独立，可以和实现标准库net/http接口的工具配合使用。

你也可以使用中间件做这些：\
\* 通过隐藏长度缓解BREACH攻击\
\* 频率限制\
\* 屏蔽恶意自动程序\
\* 提供调试信息\
\* 添加HSTS, X-Frame-Options头\
\* 从异常中优雅恢复\
\* 以及其他等等。

### 写一个简单的中间件

第一个例子中，我写了一个中间件，只允许用户从特定的域（在HTTP的Host头中有域信息）来访问服务器。这样的中间件可以保护应用程序不受“[主机欺骗攻击](http://www.skeletonscribe.net/2013/05/practical-http-host-header-attacks.html)”

### 定义类型

为了方便，让我们为这个中间件定义一种类型，叫做SingleHost。

    type SingleHost struct {

        handler     http.Handler

        allowedHost string

    }

只包含两个字段：\
\* 封装的Handler。如果是有效的Host访问，我们就调用这个Handler。\
\* 允许的主机值。\
由于我们把字段名小写了，使得该字段只对我们自己的包可见。我们还应该写一个初始化函数。

    func NewSingleHost(handler http.Handler, allowedHost string) *SingleHost {

        return &SingleHost{handler: handler, allowedHost: allowedHost}

    }

### 处理请求

现在才是实际的逻辑。为了实现http.Handler，我们的类型秩序实现一个方法：

    type Handler interface {

            ServeHTTP(ResponseWriter, *Request)

    }

这就是我们实现的方法：

    func (s *SingleHost) ServeHTTP(w http.ResponseWriter, r *http.Request) {

        host := r.Host

        if host == s.allowedHost {

            s.handler.ServeHTTP(w, r)

        } else {

            w.WriteHeader(403)

        }

    }

ServeHTTP 函数仅仅检查请求中的Host头：

-   如果Host头匹配初始化函数设置的allowedHost
    ，就调用封装handler的ServeHTTP方法。
-   如果Host头不匹配，就返回403状态码（禁止访问）。

在后一种情况中，封装handler的ServeHTTP方法根本就不会被调用。因此封装的handler根本不会有任何输出，实际上它根本就不知道有这样一个请求到来。

现在我们已经完成了自己的中间件，来把它放到应用中。这次我们不把Handler直接放到net/http服务中，而是先把Handler封装到中间件中。

    singleHosted = NewSingleHost(myHandler, "example.com")

    http.ListenAndServe(":8080", singleHosted)

### 另外一种方法

我们刚才写的中间件实在是太简单了，只有仅仅15行代码。为了写这样的中间件，引入了一个不太通用的方法。由于Go支持函数第一型和闭包，并且拥有简洁的http.HandlerFunc包装器，我们可以将其实现为一个简单的函数，而不是写一个单独的类型。下面是基于函数的中间件版本。

    func SingleHost(handler http.Handler, allowedHost string) http.Handler {

        ourFunc := func(w http.ResponseWriter, r *http.Request) {

            host := r.Host

            if host == allowedHost {

                handler.ServeHTTP(w, r)

            } else {

                w.WriteHeader(403)

            }

        }

        return http.HandlerFunc(ourFunc)

    }

这里我们声明了一个叫做SingleHost的简单函数，接受一个Handler和允许的主机名。在函数内部，我们创建了一个类似之前版本ServeHTTP的函数。这个内部函数其实是一个闭包，所以它可以从SingleHost外部访问。最终，我们通过[HandlerFunc](http://golang.org/pkg/net/http/#HandlerFunc)把这个函数用作http.Handler。

使用Handler还是定义一个http.Handler类型完全取决于你。对简单的情况而已，一个函数就足够了。但是随着中间件功能的复杂，你应该考虑定义自己的数据结构，把逻辑独立到多个方法中。

实际上，标准库这两种方法都用了。[StripPrefix](http://golang.org/pkg/net/http/#StripPrefix)
是一个返回HandlerFunc的函数。虽然[TimeoutHandler](http://golang.org/pkg/net/http/#TimeoutHandler)也是一个函数，但它返回了处理请求的自定义的类型。

### 更复杂的情况

我们的SingleHost中间件非常简单：先检查请求的一个属性，然后要么什么也不管，把请求直接传给封装的Handler；要么自己返回一个响应，根本不让封装的Handler处理这次请求。然而，有些情况是这样的，不但基于请求触发一些动作，还要在封装的Handler处理后做一些扫尾工作，比如修改响应内容等。

### 添加数据比较容易

如果我们想在封装的handler输出的内容后添加一些数据，我们只需要在handler结束后继续调用Write()即可：

    type AppendMiddleware struct {
        handler http.Handler
    }

    func (a *AppendMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
        a.handler.ServeHTTP(w, r)
        w.Write([]byte("Middleware says hello."))
    }

响应内容现在就应该包含封装的handler的内容，再加上Middleware says hello.

### 问题是

做其他的响应内容操作比较麻烦。比如，如果我们想在响应内容前写入一些数据。如果我们在封装的handler前调用Write()，那么封装的handler就好失去对HTTP状态码和HTTP头的控制。因为第一次调用Write()会直接将头输出。

想要修改原有输出（比如，替换其中的某些字符串），改变特定的HTTP头，设置不同的状态码也都因为同样的原因而不可行：当封装的handler返回时，上述数据早已被发送给客户端了。

为了处理这样的需求，我们需要一种特殊的可以用做buffer的ResponseWriter,它能够收集、暂存输出以用于修改等操作，最后再发送给客户端。我们可以将这个带buffer的ResponseWriter传给封装的handler，而不是真实的RW，这样就避免直接发送数据给客户端。

幸运的是，在Go标准库中确实存在这样一个工具。net/http/httptest中的[ResponseRecorder](http://golang.org/pkg/net/http/httptest/#ResponseRecorder)就是这样的：它保存状态码，一个保存响应头的字典，将输出累计在buffer中。尽管是用于测试（这个包名暗示了这一点），它还是很好的满足了我们的需求。

让我们看一个使用ResponseRecorder的例子，这里修改了响应内容的所有东西，是为了更完整的演示。

    type ModifierMiddleware struct {

        handler http.Handler

    }

    func (m *ModifierMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {

        rec := httptest.NewRecorder()

        // passing a ResponseRecorder instead of the original RW

        m.handler.ServeHTTP(rec, r)

        // after this finishes, we have the response recorded

        // and can modify it before copying it to the original RW

        // we copy the original headers first

        for k, v := range rec.Header() {

            w.Header()[k] = v

        }

        // and set an additional one

        w.Header().Set("X-We-Modified-This", "Yup")

        // only then the status code, as this call writes out the headers 

        w.WriteHeader(418)

        // the body hasn't been written (to the real RW) yet,

        // so we can prepend some data.

        w.Write([]byte("Middleware says hello again. "))

        // then write out the original body

        w.Write(rec.Body.Bytes())

    }

下面是我们包装的handler的输出。如果不用我们的中间件封装，原来的handler仅仅会输出Success！。

    HTTP/1.1 418 I'm a teapot

    X-We-Modified-This: Yup

    Content-Type: text/plain; charset=utf-8

    Content-Length: 37

    Date: Tue, 03 Sep 2013 18:41:39 GMT

    Middleware says hello again. Success!

这种方式提供了非常大的便利。被封装的handler现在完全在我们的控制之下：即使在其返回之后，我们也可以以任意方式操作输出。

### 和其他handlers共享数据

在不同的情况下，中间件可以需要给其他的中间件或者应用程序暴露特定的信息。比如，nosurf需要给用户提供一种获取CSRF
密钥的方式以及错误原因（如果有错误的话）。

对这种需求，一个合适的模型就是使用一个隐藏的map，将http.Request指针指向需要的数据，然后暴露一个包级别（handler级别）的函数来访问这些数据。

我在nosurf中也使用了这种模型。这里，我创建了一个全局的上下文map。注意到，由于默认情况下Go的map并不是\[并发访问安全\](<http://blog.golang.org/go-maps-in-action#TOC_6>.）的，需要一个mutex。

    type csrfContext struct {

        token string

        reason error

    }

    var (

        contextMap = make(map[*http.Request]*csrfContext)

        cmMutex    = new(sync.RWMutex)

    )

使用handler设置数据，然后通过暴露的函数Token()来获取数据。

    func Token(req *http.Request) string {

        cmMutex.RLock()

        defer cmMutex.RUnlock()

        ctx, ok := contextMap[req]

        if !ok {

                return ""

        }

        return ctx.token

    }

你可以在nosurf的代码库[context.go](https://github.com/justinas/nosurf/blob/master/context.go)中找到完整的实现。

虽然我选择在nosurf中自己实现这种需求，但实际上存在一个方便的
[gorilla/context](http://www.gorillatoolkit.org/pkg/context#ClearHandler)包，它实现了一个通用的保存请求信息的map。在大多数情况下，这个包足以满足你的需求，避免你在自己实现一个共享存储时踩坑。它甚至还有一个[自己的中间件](http://www.gorillatoolkit.org/pkg/context#ClearHandler)能在请求处理结束之后清除请求信息。

### 总结

这篇文章的目的是吸引Go用户对中间件概念的注意以及展示使用Go写中间件的一些基本组件。尽管Go是一个相对年轻的开发语言，Go拥有非常[漂亮的标准HTTP接口](http://justinas.org/embrace-gos-http-tools/)。这也是用Go写中间件是个非常简单甚至快乐的过程的原因之一。

然而，目前Go仍然缺乏高质量的HTTP工具。我之前提到的[Go中间件想法](http://justinas.org/writing-http-middleware-in-go/#usecases)，大多都还没实现。现在你已经知道如何用Go写中间件了，为什么不自己做一个呢？

PS，你可以在一个[GitHub
gist](https://gist.github.com/justinas/7059324)中找到这篇文章中所有的中间件例子。

------------------------------------------------------------------------

原文链接： [Writing HTTP Middleware in
Go](http://justinas.org/writing-http-middleware-in-go/)\
转载自： [伯乐在线](http://blog.jobbole.com/) -
[Codefor](http://blog.jobbole.com/author/codefor/)

