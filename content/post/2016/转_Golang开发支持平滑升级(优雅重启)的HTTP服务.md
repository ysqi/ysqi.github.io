
---
date: 2016-12-31T11:33:57+08:00
title: "Golang开发支持平滑升级(优雅重启)的HTTP服务"
description: ""
disqus_identifier: 1485833637636425951
slug: "Golangkai-fa-zhi-chi-ping-hua-sheng-ji-(you-ya-chong-qi-)de-HTTPfu-wu"
source: "https://segmentfault.com/a/1190000004445975"
tags: 
- golang 
categories:
- 编程语言与开发
---

原文链接：[http://tabalt.net/blog/gracef...](http://tabalt.net/blog/graceful-http-server-for-golang/)\
Golang支持平滑升级（优雅重启）的包已开源到Github：<https://github.com/tabalt/gracehttp>，欢迎使用和贡献代码。

前段时间用Golang在做一个HTTP的接口，因编译型语言的特性，修改了代码需要重新编译可执行文件，关闭正在运行的老程序，并启动新程序。对于访问量较大的面向用户的产品，关闭、重启的过程中势必会出现无法访问的情况，从而影响用户体验。

使用Golang的系统包开发HTTP服务，是无法支持平滑升级（优雅重启）的，本文将探讨如何解决该问题。

### 一、平滑升级（优雅重启）的一般思路

一般情况下，要实现平滑升级，需要以下几个步骤：

1.  用新的可执行文件替换老的可执行文件（如只需优雅重启，可以跳过这一步）

2.  通过pid给正在运行的老进程发送 特定的信号（kill -SIGUSR2 \$pid）

3.  正在运行的老进程，接收到指定的信号后，以子进程的方式启动新的可执行文件并开始处理新请求

4.  老进程不再接受新的请求，等待未完成的服务处理完毕，然后正常结束

5.  新进程在父进程退出后，会被init进程领养，并继续提供服务

### 二、Golang Socket 网络编程

Socket是程序员层面上对传输层协议TCP/IP的封装和应用。Golang中Socket相关的函数与结构体定义在net包中，我们从一个简单的例子来学习一下Golang
Socket 网络编程，关键说明直接写在注释中。

#### 1、服务端程序 server.go

    package main

    import (
        "fmt"
        "log"
        "net"
        "time"
    )

    func main() {
        // 监听8086端口
        listener, err := net.Listen("tcp", ":8086")
        if err != nil {
            log.Fatal(err)
        }
        defer listener.Close()

        for {
            // 循环接收客户端的连接，没有连接时会阻塞，出错则跳出循环
            conn, err := listener.Accept()
            if err != nil {
                fmt.Println(err)
                break
            }

            fmt.Println("[server] accept new connection.")

            // 启动一个goroutine 处理连接
            go handler(conn)
        }
    }

    func handler(conn net.Conn) {
        defer conn.Close()

        for {
            // 循环从连接中 读取请求内容，没有请求时会阻塞，出错则跳出循环
            request := make([]byte, 128)
            readLength, err := conn.Read(request)

            if err != nil {
                fmt.Println(err)
                break
            }

            if readLength == 0 {
                fmt.Println(err)
                break
            }

            // 控制台输出读取到的请求内容，并在请求内容前加上hello和时间后向客户端输出
            fmt.Println("[server] request from ", string(request))
            conn.Write([]byte("hello " + string(request) + ", time: " + time.Now().Format("2006-01-02 15:04:05")))
        }
    }

#### 2、客户端程序 client.go

    package main

    import (
        "fmt"
        "log"
        "net"
        "os"
        "time"
    )

    func main() {

        // 从命令行中读取第二个参数作为名字，如果不存在第二个参数则报错退出
        if len(os.Args) != 2 {
            fmt.Fprintf(os.Stderr, "Usage: %s name ", os.Args[0])
            os.Exit(1)
        }
        name := os.Args[1]

        // 连接到服务端的8086端口
        conn, err := net.Dial("tcp", "127.0.0.1:8086")
        checkError(err)

        for {
            // 循环往连接中 写入名字
            _, err = conn.Write([]byte(name))
            checkError(err)

            // 循环从连接中 读取响应内容，没有响应时会阻塞
            response := make([]byte, 256)
            readLength, err := conn.Read(response)
            checkError(err)

            // 将读取响应内容输出到控制台，并sleep一秒
            if readLength > 0 {
                fmt.Println("[client] server response:", string(response))
                time.Sleep(1 * time.Second)
            }
        }
    }

    func checkError(err error) {
        if err != nil {
            log.Fatal("fatal error: " + err.Error())
        }
    }

#### 3、运行示例程序

    # 运行服务端程序
    go run server.go

    # 在另一个命令行窗口运行客户端程序
    go run client.go "tabalt"

### 三、Golang HTTP 编程

HTTP是基于传输层协议TCP/IP的应用层协议。Golang中HTTP相关的实现在net/http包中，直接用到了net包中Socket相关的函数和结构体。

我们再从一个简单的例子来学习一下Golang HTTP
编程，关键说明直接写在注释中。

#### 1、http服务程序 http.go

    package main

    import (
        "log"
        "net/http"
        "os"
    )

    // 定义http请求的处理方法
    func handlerHello(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("http hello on golang\n"))
    }

    func main() {

        // 注册http请求的处理方法
        http.HandleFunc("/hello", handlerHello)

        // 在8086端口启动http服务，会一直阻塞执行
        err := http.ListenAndServe("localhost:8086", nil)
        if err != nil {
            log.Println(err)
        }

        // http服务因故停止后 才会输出如下内容
        log.Println("Server on 8086 stopped")
        os.Exit(0)
    }

#### 2、运行示例程序

    # 运行HTTP服务程序
    go run http.go

    # 在另一个命令行窗口curl请求测试页面
    curl http://localhost:8086/hello/

    # 输出如下内容：
    http hello on golang

### 四、Golang net/http包中 Socket操作的实现

从上面的简单示例中，我们看到在Golang中要启动一个http服务，只需要简单的三步：

1.  定义http请求的处理方法

2.  注册http请求的处理方法

3.  在某个端口启动HTTP服务

而最关键的启动http服务，是调用http.ListenAndServe()函数实现的。下面我们找到该函数的实现：

    func ListenAndServe(addr string, handler Handler) error {
        server := &Server{Addr: addr, Handler: handler}
        return server.ListenAndServe()
    }

这里创建了一个Server的对象，并调用它的ListenAndServe()方法，我们再找到结构体Server的ListenAndServe()方法的实现：

    func (srv *Server) ListenAndServe() error {
        addr := srv.Addr
        if addr == "" {
            addr = ":http"
        }
        ln, err := net.Listen("tcp", addr)
        if err != nil {
            return err
        }
        return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
    }

从代码上看到，这里监听了tcp端口，并将监听者包装成了一个结构体
tcpKeepAliveListener，再调用srv.Serve()方法；我们继续跟踪Serve()方法的实现：

    func (srv *Server) Serve(l net.Listener) error {
        defer l.Close()
        var tempDelay time.Duration // how long to sleep on accept failure
        for {
            rw, e := l.Accept()
            if e != nil {
                if ne, ok := e.(net.Error); ok && ne.Temporary() {
                    if tempDelay == 0 {
                        tempDelay = 5 * time.Millisecond
                    } else {
                        tempDelay *= 2
                    }
                    if max := 1 * time.Second; tempDelay > max {
                        tempDelay = max
                    }
                    srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
                    time.Sleep(tempDelay)
                    continue
                }
                return e
            }
            tempDelay = 0
            c, err := srv.newConn(rw)
            if err != nil {
                continue
            }
            c.setState(c.rwc, StateNew) // before Serve can return
            go c.serve()
        }
    }

可以看到，和我们前面Socket编程的示例代码一样，循环从监听的端口上Accept连接，如果返回了一个net.Error并且这个错误是临时性的，则会sleep一个时间再继续。
如果返回了其他错误则会终止循环。成功Accept到一个连接后，调用了方法srv.newConn()对连接做了一层包装，最后启了一个goroutine处理http请求。

### 五、Golang 平滑升级（优雅重启）HTTP服务的实现

我创建了一个新的包[gracehttp](https://github.com/tabalt/gracehttp)来实现支持平滑升级（优雅重启）的HTTP服务，为了少写代码和降低使用成本，新的包尽可能多地利用`net/http`包的实现，并和`net/http`包保持一致的对外方法。现在开始我们来看`gracehttp`包支持平滑升级
（优雅重启）Golang HTTP服务涉及到的细节如何实现。

#### 1、Golang处理信号

Golang的`os/signal`包封装了对信号的处理。简单用法请看示例：

    package main

    import (
        "fmt"
        "os"
        "os/signal"
        "syscall"
    )

    func main() {

        signalChan := make(chan os.Signal)

        // 监听指定信号
        signal.Notify(
            signalChan,
            syscall.SIGHUP,
            syscall.SIGUSR2,
        )

        // 输出当前进程的pid
        fmt.Println("pid is: ", os.Getpid())

        // 处理信号
        for {
            sig := <-signalChan
            fmt.Println("get signal: ", sig)
        }
    }

#### 2、子进程启动新程序，监听相同的端口

在第四部分的ListenAndServe()方法的实现代码中可以看到，net/http包中使用`net.Listen`函数来监听了某个端口，但如果某个运行中的程序已经监听某个端口，其他程序是无法再去监听这个端口的。解决的办法是使用子进程的方式启动，并将监听端口的文件描述符传递给子进程，子进程里从这个文件描述符实现对端口的监听。

具体实现需要借助一个环境变量来区分进程是正常启动，还是以子进程方式启动的，相关代码摘抄如下：

    // 启动子进程执行新程序
    func (this *Server) startNewProcess() error {

        listenerFd, err := this.listener.(*Listener).GetFd()
        if err != nil {
            return fmt.Errorf("failed to get socket file descriptor: %v", err)
        }

        path := os.Args[0]

        // 设置标识优雅重启的环境变量
        environList := []string{}
        for _, value := range os.Environ() {
            if value != GRACEFUL_ENVIRON_STRING {
                environList = append(environList, value)
            }
        }
        environList = append(environList, GRACEFUL_ENVIRON_STRING)

        execSpec := &syscall.ProcAttr{
            Env:   environList,
            Files: []uintptr{os.Stdin.Fd(), os.Stdout.Fd(), os.Stderr.Fd(), listenerFd},
        }

        fork, err := syscall.ForkExec(path, os.Args, execSpec)
        if err != nil {
            return fmt.Errorf("failed to forkexec: %v", err)
        }

        this.logf("start new process success, pid %d.", fork)

        return nil
    }

    func (this *Server) getNetTCPListener(addr string) (*net.TCPListener, error) {

        var ln net.Listener
        var err error

        if this.isGraceful {
            file := os.NewFile(3, "")
            ln, err = net.FileListener(file)
            if err != nil {
                err = fmt.Errorf("net.FileListener error: %v", err)
                return nil, err
            }
        } else {
            ln, err = net.Listen("tcp", addr)
            if err != nil {
                err = fmt.Errorf("net.Listen error: %v", err)
                return nil, err
            }
        }
        return ln.(*net.TCPListener), nil
    }

#### 3、父进程等待已有连接中未完成的请求处理完毕

这一块是最复杂的；首先我们需要一个计数器，在成功Accept一个连接时，计数器加1，在连接关闭时计数减1，计数器为0时则父进程可以正常退出了。Golang的sync的包里的WaitGroup可以很好地实现这个功能。

然后要控制连接的建立和关闭，我们需要深入到net/http包中Server结构体的Serve()方法。重温第四部分Serve()方法的实现，会发现如果要重新写一个Serve()方法几乎是不可能的，因为这个方法里调用了好多个不可导出的内部方法，重写Serve()方法几乎要重写整个`net/http`包。

幸运的是，我们还发现在
ListenAndServe()方法里传递了一个listener给Serve()方法，并最终调用了这个listener的Accept()方法，这个方法返回了一个Conn的示例，最终在连接断开的时候会调用Conn的Close()方法，这些结构体和方法都是可导出的！

我们可以定义自己的Listener结构体和Conn结构体，组合`net/http`包中对应的结构体，并重写Accept()和Close()方法，实现对连接的计数，相关代码摘抄如下：

    type Listener struct {
        *net.TCPListener

        waitGroup *sync.WaitGroup
    }

    func (this *Listener) Accept() (net.Conn, error) {

        tc, err := this.AcceptTCP()
        if err != nil {
            return nil, err
        }
        tc.SetKeepAlive(true)
        tc.SetKeepAlivePeriod(3 * time.Minute)

        this.waitGroup.Add(1)

        conn := &Connection{
            Conn:     tc,
            listener: this,
        }
        return conn, nil
    }

    func (this *Listener) Wait() {
        this.waitGroup.Wait()
    }

    type Connection struct {
        net.Conn
        listener *Listener

        closed bool
    }

    func (this *Connection) Close() error {

        if !this.closed {
            this.closed = true
            this.listener.waitGroup.Done()
        }

        return this.Conn.Close()
    }

#### 4、gracehttp包的用法

gracehttp包已经应用到每天几亿PV的项目中，也开源到了github上：[github.com/tabalt/gracehttp](https://github.com/tabalt/gracehttp)，使用起来非常简单。

如以下示例代码，引入包后只需修改一个关键字，将http.ListenAndServe 改为
gracehttp.ListenAndServe即可。

    package main

    import (
        "fmt"
        "net/http"

        "github.com/tabalt/gracehttp"
    )

    func main() {
        http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            fmt.Fprintf(w, "hello world")
        })

        err := gracehttp.ListenAndServe(":8080", nil)
        if err != nil {
            fmt.Println(err)
        }
    }

测试平滑升级（优雅重启）的效果，可以参考下面这个页面的说明：

<https://github.com/tabalt/gracehttp#demo>

使用过程中有任何问题和建议，欢迎[提交issue](https://github.com/tabalt/gracehttp/issues/new)反馈，也可以Fork到自己名下修改之后提交pull
request。

如果文章对您有帮助，欢迎打赏， 您的支持是我码字的动力！\

> **原文链接：[http://tabalt.net/blog/gracef...](http://tabalt.net/blog/graceful-http-server-for-golang//strong)**

