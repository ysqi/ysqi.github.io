
---
date: 2016-12-31T11:35:10+08:00
title: "深入GoPlayground内幕"
description: ""
disqus_identifier: 1485833710201085202
slug: "shen-ru--Go-Playground-nei-mu"
source: "https://segmentfault.com/a/1190000000361471"
tags: 
- golang 
topics:
- 编程语言与开发
---

### 简介

2010年9月，我们介绍了[Go
Playground](http://blog.golang.org/introducing-go-playground)，这是一个完全由Go代码组成和返回程序运行结果的web服务器。\
如果你是一位Go程序员，那你很可能已经通过阅读[Go教程](http://tour.golang.org/)或执行Go文档中的[示例](http://golang.org/pkg/strings/#pkg-examples)程序的途径使用过Go
Playground了。\
你也可以通过点击 talks.golang.org上幻灯片中的“Run”
按钮或某个博客上的程序(比如最近一篇关于[字符串](http://blog.golang.org/strings)的blog)而使用之.\
本文我们将学习Go
playground是如何实现并与其它服务整合的。其实现涉及到不同的操作系统和运行时间，这里我们假设大家用来编写Go的系统都基本相同。

### 概览

\
playground服务有三部分：\
\*
一个运行于Google服务之上的后端。它接收RPC请求，使用gc工具编译用户程序，执行，并将程序的输出（或编译错误）作为RPC响应返回。\
\* 一个运行在
[GAE](https://developers.google.com/appengine/)上的前端。它接收来自客户端的HTTP请求并生成相应的RPC请求到后端。它也做一些缓存。\
\* 一个JavaScript客户端实现的用户界面，并生成到前端的HTTP请求。

### 后端

后端程序本身很简单，所以这里我们不讨论它的实现。有趣的部分是我们如何在一个安全环境下安全地执行任意用户代码，于此同时还提供如时间、网络及文件系统等的核心功能。\
为从Google的基础设施隔离用户程序，后端将它们运行在[原生客户端](https://developers.google.com/native-client/)（或“NaCl”）中，原生客户端（NaCl）—一个Google开发的技术，允许x86程序在Web浏览器中安全执行。后端使用一个能生成NaCl可执行文件的特殊版gc工具。

（这个特殊的工具将合并到Go
1.3中。想了解更多，阅读[设计文档](http://golang.org/s/go13nacl)。如果你想提前体验NaCl，你可以[检出一个包含所有变更的分支](https://code.google.com/r/rsc-go13nacl/source/checkout)。）

本地客户端会限制程序占用CPU和RAM的使用量,此外还会阻止程序访问网络和文件系统。然而这会导致一个问题，Go程序的许多关键优势，比如并发和网络访问。此外访问文件系统，对于许多程序也是至关重要的。我们需要时间功能，才展现高效的并发性能。显然我们需要网络和文件系统，才能显示出来访问网络和文件系统方面的优势。\
尽管现在这些功能都被支持了，但是2010年发布的第一版playground时，没有一项被支持的。当前时间功能是在2009年11月10的被支持的，可是
[time.Sleep](http://golang.org/pkg/time/#Sleep)
却不能使用，而且多数与系统和网络有关的包都不被支持的\
一年后，我们在playground上面实现了一个伪时间，这才使得程序可以有个正确的休眠行为。较新的playground更新引入了伪网络和伪文件系统，这使得playground的工具链与正常的Go工具链相同。这些新引入的功能会在下面具体阐述。

### 伪时间

playground里面的程序可用CPU时间和内存都是有限的。除此以外程序实际使用时间也是有限制的。这是因为每个运行在playground的程序都消耗着后台资源，以及占据客户端和后台间的基础设施。限制每个程序的运行时间让我们的维护更加可遇见，而且可以保护我们免受拒绝服务攻击。\
但是当程序使用时间功能函数的时候，这些限制将变得非常不合适。在 [Go
Concurrency Patterns](http://talks.golang.org/2012/concurrency.slide)
讲话中通过一个例子来演示这个糟糕的问题。这是一个使用时间功能函数比如
[time.Sleep](http://golang.org/pkg/time/#Sleep)
和[time.After](http://golang.org/pkg/time/#After)的例子程序，当运行在早期的playground中时，这些程序的休眠会失效而且行为很奇怪（有时甚至出现错误）

通过使用一个高明的小把戏，我们可以使得Go程序认为它是在休眠，而实际上这个休眠没有花费任何时间。在介绍这个小把戏之前，我们需要了解调度程序是管理goroutine的休眠的原理。\
当一个goroutine调用time.Sleep（或者其他相似函数），调度器会在挂起的计时器堆中添加中增加一个计时器，并让goroutine休眠。在这期间，一个特殊的goroutine计算器管理着这个堆。当这个特殊的goroutine计算器开始工作时，首先，它告诉调度器，当堆中的下一个挂起的计时器准备计时的时候唤醒自己，然后它自己就开始休眠了。当这个特殊计时器被唤醒后首先是检测是否有计时器超时了，如果有那么就唤醒相应的goroutine，然后又回到休眠状态。\
明白了这个原理后，那个小把戏只是改变唤醒goroutine的计时器的条件。调度器并不是经过一段时间后进行唤醒，而且仅仅等待一个所有goroutines
都阻塞的死锁产生后就进行唤醒。

playground运行时版本中维护着一个内部时钟。当修改后的调度器检测到一个死锁，那么它将检查是否有一些挂起的计时器。如果有的话，它会将内部时钟的时间调整到最早计时器的促发时间，然后唤醒goroutine计时器。这样一直循环往复，程序都认为时间过去了，而实际上休眠几乎没有耗时。\
这些调度器的改变细节详见
[proc.c](https://code.google.com/r/rsc-go13nacl/source/diff?spec=svnc9a5be0fa2db5edcf8f8788da9f7eed323df6c07&r=c9a5be0fa2db5edcf8f8788da9f7eed323df6c07&format=side&path=/src/pkg/runtime/proc.c#sc_svn35d5bae6aac826e6db1851ea14fe9f8da95088a8_2314)
和
[time.goc](https://code.google.com/r/rsc-go13nacl/source/diff?spec=svnc9a5be0fa2db5edcf8f8788da9f7eed323df6c07&r=c9a5be0fa2db5edcf8f8788da9f7eed323df6c07&format=side&path=/src/pkg/runtime/time.goc#sc_svnc9a5be0fa2db5edcf8f8788da9f7eed323df6c07_178)。\
伪时间解决了后台资源耗尽的问题，但是程序的输出该怎么办呢？看见一个在休眠的程序，却几乎不耗时地正确完成工作了，这是得多么的奇怪啊！

下面的程序每秒输出当前时间，然后三秒后退出.试着运行一下。

    func main() {
        stop := time.After(3 * time.Second)
        tick := time.NewTicker(1 * time.Second)
        defer tick.Stop()
        for {
            select {
            case <-tick.C:
                fmt.Println(time.Now())
            case <-stop:
                return
            }
        }
    }

这是如何做到的? 这其实是后台、前端和客户端合作的结果。\
我们捕获到每次向标准输出和标准错误输出的时间，并把这个时间提供给客户端。那么客户端就可以以正确的时间间隔输出，以至于这个输出就像是本地程序输出的一样。

playground的运行环境包提供了一个在每个写入数据之前引入一个小“回放头”的特殊[写函数](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/runtime/sys_nacl_amd64p32.s?r=1f01be1a1dc2#54)，它。回放头中包含一个逻辑字符，当前时间，要写入数据长度。一个写操作的回放头结构如下：

    0 0 P B <8-byte time> <4-byte data length> <data>

这个程序的原始输出类似这样：

    \x00\x00PB\x11\x74\xef\xed\xe6\xb3\x2a\x00\x00\x00\x00\x1e2009-11-10 23:00:01 +0000 UTC
    \x00\x00PB\x11\x74\xef\xee\x22\x4d\xf4\x00\x00\x00\x00\x1e2009-11-10 23:00:02 +0000 UTC
    \x00\x00PB\x11\x74\xef\xee\x5d\xe8\xbe\x00\x00\x00\x00\x1e2009-11-10 23:00:03 +0000 UTC

前端将这些输出解析为一系列事件并返回给客户端一个事件列表的JSON对象：

    {
        "Errors": "",
        "Events": [
            {
                "Delay": 1000000000,
                "Message": "2009-11-10 23:00:01 +0000 UTC\n"
            },
            {
                "Delay": 1000000000,
                "Message": "2009-11-10 23:00:02 +0000 UTC\n"
            },
            {
                "Delay": 1000000000,
                "Message": "2009-11-10 23:00:03 +0000 UTC\n"
            }
        ]
    }

JavaScript客户端（在用户的Web浏览器中运行的）然后使用提供的延迟间隔回放这个事件。对用户来说看起来程序是在实时运行。

### 伪文件系统

在Go本地客户端（NaCl）的工具链上构建的程序，是不能访问本地机器的文件系统的。为了解决这个问题syscall包中有个文件访问的函数（Open,
Read,
Write等等）都是操作在一个内存文件系统上的。这个内存文件系统是由syscall包自身实现的。既然syscall包是一个Go代码与操作系统内存间的一个接口，那么用户程序会将这个伪文件系统会和一个真实的文件系统一个样看待。\
下面的示例程序将数据写入一个文件，让后复制内容到标准输出。试着运行一下（你也可以进行编辑）

    func main() {
        const filename = "/tmp/file.txt"

        err := ioutil.WriteFile(filename, []byte("Hello, file system\n"), 0644)
        if err != nil {
            log.Fatal(err)
        }

        b, err := ioutil.ReadFile(filename)
        if err != nil {
            log.Fatal(err)
        }

        fmt.Printf("%s", b)
    }

当一个进程开始，这个伪文件系统加入/dev目录下的设备和一个/tmp空目录。那么程序可以对这个文件系统和平常一样进行操作，但是进程退出后，所有对文件系统的改变将会丢失

在初始化的时候，可以上传zip压缩文件（详见[unzip\_nacl.go](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/unzip_nacl.go)）迄今为止只会在进行标准库测试的时候，我们会使用解压缩工具来提供测试数据文件。可是我们打算playground程序可以运行文档示例、博客帖子和Golang的教程里面的数据。\
具体实现详见
[fs\_nacl.go](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/fs_nacl.go)
和
[fd\_nacl.go](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/fd_nacl.go)
文件(由于是\_nacl的后缀，所以只有当GOOS被设置为nacl时候，这些文件才会被加入到syscall包中）。\
这个伪文件系统由 [fsys
struct](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/fs_nacl.go?r=5317d308abe4078f68aecc14fd5fb95303d62a06#25)
代表。其中一个全局实例（称为fs）在初始化的时候被创建。各种和文件有关的函数都操作在fs上，而不是进行真实的系统调用。例如，这里有个
[syscall.Open](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/fs_nacl.go?r=1f01be1a1dc2#467)函数：

    func Open(path string, openmode int, perm uint32) (fd int, err error) {
        fs.mu.Lock()
        defer fs.mu.Unlock()
        f, err := fs.open(path, openmode, perm&0777|S_IFREG)
        if err != nil {
            return -1, err
        }
        return newFD(f), nil
    }

文件描述符被一个称为files的全局片段记录着。每个文件描述符对应着一个file，而且每个file都会提供一
fileImpl接口的实现。这里有几个接口的实现：\
\* fsysFile代表常规文件和设备 (such as/dev/random) ,\
\*
标准输入输出和标准错误都是naclFile的实例，这可以使用系统调用来操作真实文件（这是playground中的程序唯一访问外部环境的途径，\
\* 网络套接字有着自己的实现，下面章节中会讨论.

### 伪网络访问

和文件系统一样，playground的网络堆栈是由syscall包在进程内部模拟出来的,这可以让playground项目使用回送地址（127.0.0.1）。但不能请求其他主机。

运行下面可执行的实例代码。这个程序首先会监听TCP的端口，接着等待连接的到来，然后将连接传来的数据复制到标准输出，最后程序退出。在另外一个goroutine中，他会连接那个监听中的端口，然后向连接里面写入数据，最后关闭。

    func main() {
        l, err := net.Listen("tcp", "127.0.0.1:4000")
        if err != nil {
            log.Fatal(err)
        }
        defer l.Close()

        go dial()

        c, err := l.Accept()
        if err != nil {
            log.Fatal(err)
        }
        defer c.Close()

        io.Copy(os.Stdout, c)
    }

    func dial() {
        c, err := net.Dial("tcp", "127.0.0.1:4000")
        if err != nil {
            log.Fatal(err)
        }
        defer c.Close()
        c.Write([]byte("Hello, network\n"))
    }

网络的接口比文件要复杂的多，所以伪网络的接口的实现会比伪文件系统的要庞大和复杂的多。伪网络必须模拟读和写的超时，以及处理不同地址类型和协议等等。\
具体实现详见[net\_nacl.go](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/net_nacl.go?r=1f01be1a1dc2)。推荐从[netFile](https://code.google.com/r/rsc-go13nacl/source/browse/src/pkg/syscall/net_nacl.go?r=1f01be1a1dc2#419)开始阅读，因为这是网络套接字对于fileImpl接口的实现。

### 前端

playground的前端是另外一个简单的程序 (不到100行).
它的主要功能是接受客户端的HTTP请求，然后向后台发出对应的RPC请求，同时还会完成一些缓存工作。\
前端提供一个HTTP处理程序，详见<http://golang.org/compile>。这个处理程序接受带有body标签（其中包含要运行的Go程序代码）和一个可选version标签（多数客户端应该是‘2’）的POST请求。\
当前端收到一个HTTP编译请求的时候，它首先查看缓存，检查之前是否有过同样的编译请求。如果发现存在同，那么就会将缓存的响应直接返回。缓存可以防止像Go主页上那样的大众化程序让后台过载。如果发现该请求之前没有被缓存过，那么前端会向后台发出相应的RPC请求，然后缓存后台的响应，接着分析对应的事件回放（详见伪时间），最后通过HTTP响应将JSON格式的对象返回到客户端（像上面描述那样）。

### 客户端

各种使用playground的站点，共享着一些同样的Javascript代码来搭建用户访问接口（代码窗口和输出窗口，运行按钮等等），通过这些接口来后playground前端交互。\
具体实现在go.tool资源库的[playground.js](https://code.google.com/p/go/source/browse/godoc/static/playground.js?repo=tools)文件中，可以通过[go.tools/godoc/static](http://godoc.org/code.google.com/p/go.tools/godoc/static)包来导入。
其中一些代码较为简洁，也有一些比较繁杂,
因为这是由几个不同的客户端代码合并出来的。\
[playground](https://code.google.com/p/go/source/browse/godoc/static/playground.js?repo=tools#226)函数使用一些HTML元素，然后构成一个交互式的playground窗口小部件。如果你想将playground添加到你的站点的话，你就可以使用这些函数。\
[Transport](https://code.google.com/p/go/source/browse/godoc/static/playground.js?repo=tools#6)接口
(非正式的定义, 是JavaScript脚本)的设计是依据网站前端交互方式提。
[HTTPTransport](https://code.google.com/p/go/source/browse/godoc/static/playground.js?repo=tools#43)是一个Transport的实现，可以发送如前描述的以HTTP为基础的协议。
[SocketTransport](https://code.google.com/p/go/source/browse/godoc/static/playground.js?repo=tools#115)是另外一个实现，发送WebSocket
(详见下面的'Playing offline')。\
为了遵守\[同源策略\](<http://en.wikipedia.org/wiki/Same-origin_policy>），各种网站服务器（例如godoc）通过playground在<http://golang.org/compile>下的服务来完成代理请求。这个代理是通过共有的
[go.tools/playground](http://godoc.org/code.google.com/p/go.tools/playground)
包来完成的。

### 离线运行

不管是[Go Tour](http://tour.golang.org/)还是[Present
Tool](http://godoc.org/code.google.com/p/go.talks/present)都可以离线运行。
这样的离线功能对于访问网络有限制的人们来说，实在太棒了。\
为了离线运行，这些工具在本地运行一个特殊版本的playground后端。这个特殊的后端使用的是常规GO\
工具，这些工具没有上面提到的那些修改，而且使用WebSocker来与客户端进行通信。\
WebSocket的后端实现详见[go.tools/playground/socket](http://godoc.org/code.google.com/p/go.tools/playground/socket)包。在[Inside
Present](http://talks.golang.org/2012/insidepresent.slide#1)讲话中讨论了代码细节。

### 其他客户端

playground服务不单单只有为了给Go项目官方使用 ([Go by
Example](https://gobyexample.com/)是另外一个例子)
。我们很高兴你能在你的站点使用该服务。我们唯一的要求就是您事先和我们联系，在您的请求中使用唯一用户代理（这样我们可以确认您的身份），此外您提供的服务是有益于Go社区的。

### 结束语

不论是godoc，是tour，还是这样的blog，playground已经成为Go文档系列中不可或缺的一部分了。随着最近的伪文件系统和伪网络堆栈的引入，我们将激动地完善我们的学习资料来覆盖这些新内容。\
但是，最后，playground只是冰山一角，随着本地客户端（Native
Client）将要支持Go1.3，我们期盼着社区做出更棒的功能。

这篇文章是12月12号的[Go Advent
Calendar](http://blog.gopheracademy.com/go-advent-2013)中的一篇，[Go
AdventCalendar](http://blog.gopheracademy.com/go-advent-2013)是一系列的博客帖子集合。

作者 Andrew Gerrand

### 相关文章

-   [Learn Go from your
    browser](http://blog.golang.org/learn-go-from-your-browser)
-   [Introducing the Go
    Playground](http://blog.golang.org/introducing-go-playground)

------------------------------------------------------------------------

原文：[Inside the Go Playground](http://blog.golang.org/playground)\
转载自：[开源中国社区](http://www.oschina.net/translate/inside-the-go-playground)--[Mitisky](http://my.oschina.net/u/1032750),
[Garfielt](http://my.oschina.net/Garfielt),
[cmy00cmy](http://my.oschina.net/u/1385461),
[JAVA草根](http://my.oschina.net/u/1044431)

