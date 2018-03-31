
---
date: 2016-12-31T11:34:18+08:00
title: "Go标准库剖析1(transporthttp请求的承载者)"
description: ""
disqus_identifier: 1485833658836237616
slug: "Go-biao-zhun-ku-pou-xi--1(transport-http-qing-qiu-de-cheng-zai-zhe-)"
source: "https://segmentfault.com/a/1190000003735562"
tags: 
- http 
- 框架源码 
- 读源码 
- 源码分析 
- golang 
categories:
- 编程语言与开发
---

使用golang net/http库发送http请求，最后都是调用 transport的
RoundTrip方法

    type RoundTripper interface {
        RoundTrip(*Request) (*Response, error)
    }

`RoundTrip executes a single HTTP transaction, returning the Response for the request req.`
(RoundTrip 代表一个http事务，给一个请求返回一个响应)\
说白了，就是你给它一个request,它给你一个response

下面我们来看一下他的实现，对应源文件`net/http/transport.go`，我感觉这里是http
package里面的精髓所在，go里面一个struct就跟一个类一样，transport这个类长这样的

    type Transport struct {
        idleMu     sync.Mutex
        wantIdle   bool // user has requested to close all idle conns
        idleConn   map[connectMethodKey][]*persistConn
        idleConnCh map[connectMethodKey]chan *persistConn

        reqMu       sync.Mutex
        reqCanceler map[*Request]func()

        altMu    sync.RWMutex
        altProto map[string]RoundTripper // nil or map of URI scheme => RoundTripper
        //Dial获取一个tcp 连接，也就是net.Conn结构，你就记住可以往里面写request
        //然后从里面搞到response就行了
        Dial func(network, addr string) (net.Conn, error)
    }

篇幅所限， https和代理相关的我就忽略了， 两个 `map` 为
`idleConn`、`idleConnCh`，`idleConn` 是保存从 connectMethodKey
（代表着不同的协议 不同的host，也就是不同的请求）到 persistConn 的映射，
`idleConnCh` 用来在并发http请求的时候在多个 goroutine
里面相互发送持久连接，也就是说， 这些持久连接是可以重复利用的，
你的http请求用某个`persistConn`用完了，通过这个`channel`发送给其他http请求使用这个`persistConn`，然后我们找到`transport`的`RoundTrip`方法

    func (t *Transport) RoundTrip(req *Request) (resp *Response, err error) {
        ...
        pconn, err := t.getConn(req, cm)
        if err != nil {
            t.setReqCanceler(req, nil)
            req.closeBody()
            return nil, err
        }

        return pconn.roundTrip(treq)
    }

前面对输入的错误处理部分我们忽略，
其实就2步，先获取一个TCP长连接，所谓TCP长连接就是三次握手建立连接后不`close`而是一直保持重复使用（节约环保）
然后调用这个持久连接persistConn 这个struct的roundTrip方法

我们跟踪第一步

    func (t *Transport) getConn(req *Request, cm connectMethod) (*persistConn, error) {
        if pc := t.getIdleConn(cm); pc != nil {
            // set request canceler to some non-nil function so we
            // can detect whether it was cleared between now and when
            // we enter roundTrip
            t.setReqCanceler(req, func() {})
            return pc, nil
        }
     
        type dialRes struct {
            pc  *persistConn
            err error
        }
        dialc := make(chan dialRes)
        //定义了一个发送 persistConn的channel

        prePendingDial := prePendingDial
        postPendingDial := postPendingDial

        handlePendingDial := func() {
            if prePendingDial != nil {
                prePendingDial()
            }
            go func() {
                if v := <-dialc; v.err == nil {
                    t.putIdleConn(v.pc)
                }
                if postPendingDial != nil {
                    postPendingDial()
                }
            }()
        }

        cancelc := make(chan struct{})
        t.setReqCanceler(req, func() { close(cancelc) })
     
        // 启动了一个goroutine, 这个goroutine 获取里面调用dialConn搞到
        // persistConn, 然后发送到上面建立的channel  dialc里面，    
        go func() {
            pc, err := t.dialConn(cm)
            dialc <- dialRes{pc, err}
        }()

        idleConnCh := t.getIdleConnCh(cm)
        select {
        case v := <-dialc:
            // dialc 我们的 dial 方法先搞到通过 dialc通道发过来了
            return v.pc, v.err
        case pc := <-idleConnCh:
            // 这里代表其他的http请求用完了归还的persistConn通过idleConnCh这个    
            // channel发送来的
            handlePendingDial()
            return pc, nil
        case <-req.Cancel:
            handlePendingDial()
            return nil, errors.New("net/http: request canceled while waiting for connection")
        case <-cancelc:
            handlePendingDial()
            return nil, errors.New("net/http: request canceled while waiting for connection")
        }
    }

这里面的代码写的很有讲究 , 上面代码里面我也注释了， 定义了一个发送
`persistConn`的channel` dialc`， 启动了一个`goroutine`, 这个`goroutine`
获取里面调用`dialConn`搞到`persistConn`,
然后发送到`dialc`里面，主协程`goroutine`在
`select`里面监听多个`channel`,看看哪个通道里面先发过来
`persistConn`，就用哪个，然后`return`。

这里要注意的是 `idleConnCh`
这个通道里面发送来的是其他的http请求用完了归还的`persistConn`，
如果从这个通道里面搞到了，`dialc`这个通道也等着发呢，不能浪费，就通过`handlePendingDial`这个方法把`dialc`通道里面的`persistConn`也发到`idleConnCh`，等待后续给其他http请求使用。

还有就是，读者可以翻一下代码，每个新建的persistConn的时候都把tcp连接里地输入流，和输出流用br（`br       *bufio.Reader`）,和bw(`bw *bufio.Writer`)包装了一下，往bw写就写到tcp输入流里面了，读输出流也是通过br读，并启动了读循环和写循环

    pconn.br = bufio.NewReader(noteEOFReader{pconn.conn, &pconn.sawEOF})
    pconn.bw = bufio.NewWriter(pconn.conn)
    go pconn.readLoop()
    go pconn.writeLoop()

我们跟踪第二步`pconn.roundTrip` 调用这个持久连接persistConn
这个struct的`roundTrip`方法。\
先瞄一下 `persistConn` 这个struct

    type persistConn struct {
        t        *Transport
        cacheKey connectMethodKey
        conn     net.Conn
        tlsState *tls.ConnectionState
        br       *bufio.Reader       // 从tcp输出流里面读
        sawEOF   bool                // whether we've seen EOF from conn; owned by readLoop
        bw       *bufio.Writer       // 写到tcp输入流
         reqch    chan requestAndChan // 主goroutine 往channnel里面写，读循环从     
                                     // channnel里面接受
        writech  chan writeRequest   // 主goroutine 往channnel里面写                                      
                                     // 写循环从channel里面接受
        closech  chan struct{}       // 通知关闭tcp连接的channel 
        
        writeErrCh chan error

        lk                   sync.Mutex // guards following fields
        numExpectedResponses int
        closed               bool // whether conn has been closed
        broken               bool // an error has happened on this connection; marked broken so it's not reused.
        canceled             bool // whether this conn was broken due a CancelRequest
        // mutateHeaderFunc is an optional func to modify extra
        // headers on each outbound request before it's written. (the
        // original Request given to RoundTrip is not modified)
        mutateHeaderFunc func(Header)
    }

里面是各种channel, 用的是出神入化， 各位要好好理解一下， 我这里画一下

这里有三个goroutine，分别用三个圆圈表示， channel用箭头表示

有两个channel `writeRequest` 和 `requestAndChan`

    type writeRequest struct {
        req *transportRequest
        ch  chan<- error
    }

主goroutine 往writeRequest里面写，写循环从writeRequest里面接受

    type responseAndError struct {
        res *Response
        err error
    }

    type requestAndChan struct {
        req *Request
        ch  chan responseAndError
        addedGzip bool
    }

主goroutine 往requestAndChan里面写，读循环从requestAndChan里面接受。

注意这里的channel都是双向channel，也就是channel
的struct里面有一个chan类型的字段， 比如 `reqch    chan requestAndChan`
这里的 requestAndChan 里面的 `ch  chan responseAndError`。

这个是很牛叉，主 goroutine 通过 reqch 发送requestAndChan
给读循环，然后读循环搞到response后通过 requestAndChan
里面的通道responseAndError把response返给主goroutine，所以我画了一个双向箭头。

我们研究一下代码，我理解下来其实就是三个goroutine通过channel互相协作的过程。

主循环：

    func (pc *persistConn) roundTrip(req *transportRequest) (resp *Response, err error) {
        ... 忽略
        // Write the request concurrently with waiting for a response,
        // in case the server decides to reply before reading our full
        // request body.
        writeErrCh := make(chan error, 1)
        pc.writech <- writeRequest{req, writeErrCh}
        //把request发送给写循环
        resc := make(chan responseAndError, 1)
        pc.reqch <- requestAndChan{req.Request, resc, requestedGzip}
        //发送给读循环
        var re responseAndError
        var respHeaderTimer <-chan time.Time
        cancelChan := req.Request.Cancel
    WaitResponse:
        for {
            select {
            case err := <-writeErrCh:
                if isNetWriteError(err) {
                    //写循环通过这个channel报告错误
                    select {
                    case re = <-resc:
                        pc.close()
                        break WaitResponse
                    case <-time.After(50 * time.Millisecond):
                        // Fall through.
                    }
                }
                if err != nil {
                    re = responseAndError{nil, err}
                    pc.close()
                    break WaitResponse
                }
                if d := pc.t.ResponseHeaderTimeout; d > 0 {
                    timer := time.NewTimer(d)
                    defer timer.Stop() // prevent leaks
                    respHeaderTimer = timer.C
                }
            case <-pc.closech:
                // 如果长连接挂了， 这里的channel有数据， 进入这个case, 进行处理
                
                select {
                case re = <-resc:
                    if fn := testHookPersistConnClosedGotRes; fn != nil {
                        fn()
                    }
                default:
                    re = responseAndError{err: errClosed}
                    if pc.isCanceled() {
                        re = responseAndError{err: errRequestCanceled}
                    }
                }
                break WaitResponse
            case <-respHeaderTimer:
                pc.close()
                re = responseAndError{err: errTimeout}
                break WaitResponse
                // 如果timeout，这里的channel有数据， break掉for循环
            case re = <-resc:
                break WaitResponse
               // 获取到读循环的response, break掉 for循环
            case <-cancelChan:
                pc.t.CancelRequest(req.Request)
                cancelChan = nil
            }
        }

        if re.err != nil {
            pc.t.setReqCanceler(req.Request, nil)
        }
        return re.res, re.err
    }

这段代码主要就干了三件事

-   主goroutine -&gt;requestAndChan -&gt; 读循环goroutine

-   主goroutine -&gt;writeRequest-&gt; 写循环goroutine

-   主goroutine 通过select 监听各个channel上的数据， 比如请求取消，
    timeout，长连接挂了，写流出错，读流出错， 都是其他goroutine
    发送过来的，
    跟中断一样，然后相应处理，上面也提到了，有些channel是主goroutine通过channel发送给其他goroutine的struct里面包含的channel,
    比如 `case err := <-writeErrCh:` `case re = <-resc:`

读循环代码：

    func (pc *persistConn) readLoop() {
        
        ... 忽略
        alive := true
        for alive {
            
            ... 忽略
            rc := <-pc.reqch

            var resp *Response
            if err == nil {
                resp, err = ReadResponse(pc.br, rc.req)
                if err == nil && resp.StatusCode == 100 {
                    //100  Continue  初始的请求已经接受，客户应当继续发送请求的其 
                    // 余部分
                    resp, err = ReadResponse(pc.br, rc.req)
                    // 读pc.br（tcp输出流）中的数据，这里的代码在response里面
                    //解析statusCode，头字段， 转成标准的内存中的response 类型
                    //  http在tcp数据流里面，head和body以 /r/n/r/n分开， 各个头
                    // 字段 以/r/n分开
                }
            }

            if resp != nil {
                resp.TLS = pc.tlsState
            }

            ...忽略
            //上面处理一些http协议的一些逻辑行为，
            rc.ch <- responseAndError{resp, err} //把读到的response返回给    
                                                 //主goroutine

            .. 忽略
            //忽略部分， 处理cancel req中断， 发送idleConnCh归还pc（持久连接）到持久连接池中（map）    
        pc.close()
    }

无关代码忽略，这段代码主要干了一件事情

> 读循环goroutine 通过channel requestAndChan
> 接受主goroutine发送的request(`rc := <-pc.reqch`),
> 并从tcp输出流中读取response， 然后反序列化到结构体中， 最后通过channel
> 返给主goroutine (`rc.ch <- responseAndError{resp, err} `)

    func (pc *persistConn) writeLoop() {
        for {
            select {
            case wr := <-pc.writech:   //接受主goroutine的 request
                if pc.isBroken() {
                    wr.ch <- errors.New("http: can't write HTTP request on broken connection")
                    continue
                }
                err := wr.req.Request.write(pc.bw, pc.isProxy, wr.req.extra)   //写入tcp输入流
                if err == nil {
                    err = pc.bw.Flush()
                }
                if err != nil {
                    pc.markBroken()
                    wr.req.Request.closeBody()
                }
                pc.writeErrCh <- err 
                wr.ch <- err         //  出错的时候返给主goroutineto 
            case <-pc.closech:
                return
            }
        }
    }

写循环就更简单了，select
channel中主gouroutine的request，然后写入tcp输入流，如果出错了，channel
通知调用者。

整体看下来，过程都很简单，但是代码中有很多值得我们学习的地方，比如高并发请求如何复用tcp连接，这里是连接池的做法，如果使用多个
goroutine相互协作完成一个http请求，出现错误的时候如何通知调用者中断错误，代码风格也有很多可以借鉴的地方。

我打算写一个系列，全面剖析go标准库里面的精彩之处，分享给大家。

