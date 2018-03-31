
---
date: 2017-05-24T09:17:31+08:00
title: "fasthttp中的协程池实现"
description: ""
disqus_identifier: 1495588651423248752
slug: "fasthttpzhong-de-xie-cheng-chi-shi-xian"
source: "https://segmentfault.com/a/1190000009133154"
tags: 
- goroutine 
- web 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

fasthttp中的协程池实现
======================

> 协程池可以控制并行度，复用协程。fasthttp 比 net/http
> 效率高很多倍的重要原因，就是利用了协程池。实现并不复杂，我们可以参考他的设计，写出高性能的应用。

入口
----

    // server.go

    func (s *Server) Serve(ln net.Listener) error {
        var lastOverflowErrorTime time.Time
        var lastPerIPErrorTime time.Time
        var c net.Conn
        var err error

        maxWorkersCount := s.getConcurrency()
        s.concurrencyCh = make(chan struct{}, maxWorkersCount)
        wp := &workerPool{
            WorkerFunc:      s.serveConn,
            MaxWorkersCount: maxWorkersCount,
            LogAllErrors:    s.LogAllErrors,
            Logger:          s.logger(),
        }
        // break-00
        wp.Start()

        for {
            // break-02
            if c, err = acceptConn(s, ln, &lastPerIPErrorTime); err != nil {
                wp.Stop()
                if err == io.EOF {
                    return nil
                }
                return err
            }

            // break-03
            if !wp.Serve(c) {
                s.writeFastError(c, StatusServiceUnavailable,
                    "The connection cannot be served because Server.Concurrency limit exceeded")
                c.Close()
                if time.Since(lastOverflowErrorTime) > time.Minute {
                    s.logger().Printf("The incoming connection cannot be served, because %d concurrent connections are served. "+
                        "Try increasing Server.Concurrency", maxWorkersCount)
                    lastOverflowErrorTime = CoarseTimeNow()
                }

                // The current server reached concurrency limit,
                // so give other concurrently running servers a chance
                // accepting incoming connections on the same address.
                //
                // There is a hope other servers didn't reach their
                // concurrency limits yet :)
                time.Sleep(100 * time.Millisecond)
            }
            c = nil
        }
    }

    // 有必要了解一下 workerPool 的结构
    type workerPool struct {
        // Function for serving server connections.
        // It must leave c unclosed.
        WorkerFunc func(c net.Conn) error

        MaxWorkersCount int

        LogAllErrors bool

        MaxIdleWorkerDuration time.Duration

        Logger Logger

        lock         sync.Mutex
        workersCount int
        mustStop     bool

        ready []*workerChan

        stopCh chan struct{}

        workerChanPool sync.Pool
    }

goroutine status:

1.  main0: wp.Start()

break-00
--------

    // workerpool.go

    // 启动一个 goroutine， 每隔一段时间，清理一下 []*workerChan; 
    // wp.clean() 的操作是 查看最近使用的workerChan, 如果他的最近使用间隔大于某个值，那么把这个workerChan清理了。
    func (wp *workerPool) Start() {
        if wp.stopCh != nil {
            panic("BUG: workerPool already started")
        }
        wp.stopCh = make(chan struct{})
        stopCh := wp.stopCh
        go func() {
            var scratch []*workerChan
            for {
                // break-01
                wp.clean(&scratch)
                select {
                case <-stopCh:
                    return
                default:
                    time.Sleep(wp.getMaxIdleWorkerDuration())
                }
            }
        }()
    }

goroutine status:

1.  main0: wp.Start()

2.  g1: for loop to clean idle workerChan

break-01
--------

    func (wp *workerPool) clean(scratch *[]*workerChan) {
        maxIdleWorkerDuration := wp.getMaxIdleWorkerDuration()

        // Clean least recently used workers if they didn't serve connections
        // for more than maxIdleWorkerDuration.
        currentTime := time.Now()

        wp.lock.Lock()
        ready := wp.ready
        n := len(ready)
        i := 0
        // 这里从队列头部取出超过 最大空闲时间 的workerChan。
        // 可以看出，最后使用的workerChan 一定是放回队列尾部的。
        for i < n && currentTime.Sub(ready[i].lastUseTime) > maxIdleWorkerDuration {
            i++
        }
        // 把空闲的放入 scratch, 剩余的放回 ready
        *scratch = append((*scratch)[:0], ready[:i]...)
        if i > 0 {
            m := copy(ready, ready[i:])
            for i = m; i < n; i++ {
                ready[i] = nil
            }
            wp.ready = ready[:m]
        }
        wp.lock.Unlock()

        // Notify obsolete workers to stop.
        // This notification must be outside the wp.lock, since ch.ch
        // may be blocking and may consume a lot of time if many workers
        // are located on non-local CPUs.
        tmp := *scratch
        // 销毁的操作就是向 chan net.Conn 中塞入一个 nil, 后面会看到解释
        for i, ch := range tmp {
            ch.ch <- nil
            tmp[i] = nil
        }
    }

break-02
--------

`acceptConn(s, ln, &lastPerIPErrorTime)` 主要处理
ln.Accept()，判断err是否是 Temporary 的，最终返回一个 net.Conn

break-03
--------

    // workerpool.go

    func (wp *workerPool) Serve(c net.Conn) bool {
        // break-04
        ch := wp.getCh()
        if ch == nil {
            return false
        }
        ch.ch <- c
        return true
    }

    type workerChan struct {
        lastUseTime time.Time
        ch          chan net.Conn
    }

wp.getCh() 返回一个 \*workerChan, 可以看到， workerChan 有一个 ch
属性，参数传入的 net.Conn 直接往里面塞。

break-04
--------

    // workerpool.go

    func (wp *workerPool) getCh() *workerChan {
        var ch *workerChan
        createWorker := false

        wp.lock.Lock()
        ready := wp.ready
        n := len(ready) - 1
        if n < 0 {
            // ready 为空，并且总数小于 MaxWorkersCount，那么需要创建新的 workerChan
            if wp.workersCount < wp.MaxWorkersCount {
                createWorker = true
                wp.workersCount++
            }
        } else {
            // 从队列尾部取出一个 workerChan
            ch = ready[n]
            ready[n] = nil
            wp.ready = ready[:n]
        }
        wp.lock.Unlock()

        if ch == nil {
            if !createWorker {
                return nil
            }
            // 走入创建流程，从 Pool中取出 workerChan
            vch := wp.workerChanPool.Get()
            if vch == nil {
                vch = &workerChan{
                    ch: make(chan net.Conn, workerChanCap),
                }
            }
            ch = vch.(*workerChan)
            // 创建goroutine处理请求，接收一个 chan *workerChan 作为参数
            go func() {
                // break-05
                wp.workerFunc(ch)
                wp.workerChanPool.Put(vch)
            }()
        }
        return ch
    }

这里我们只看创建的流程。如果ready为空，说明ready被耗尽，并且小于
MaxWorkersCount，那么需要创建新的 workerChan。\
创建时，先从 Pool 中取出复用，如果为nil，再创建新的。\
可以预测到，这里 wp.workerFunc(ch) 必定包含一个 for 循环，处理
workerChan 中的 net.Conn。

goroutine status:

1.  main0: wp.Start()

2.  g1: for loop to clean idle workerChan

3.  g2: wp.workerFunc(ch) blocks for handling connection

break-05
--------

    // workerpool.go

    func (wp *workerPool) workerFunc(ch *workerChan) {
        var c net.Conn

        var err error
        for c = range ch.ch {
            if c == nil {
                break
            }

            // 正真的处理请求的函数
            if err = wp.WorkerFunc(c); err != nil && err != errHijacked {
                errStr := err.Error()
                if wp.LogAllErrors || !(strings.Contains(errStr, "broken pipe") ||
                    strings.Contains(errStr, "reset by peer") ||
                    strings.Contains(errStr, "i/o timeout")) {
                    wp.Logger.Printf("error when serving connection %q<->%q: %s", c.LocalAddr(), c.RemoteAddr(), err)
                }
            }
            if err != errHijacked {
                c.Close()
            }
            c = nil

            // 释放 workerChan
            // break-06
            if !wp.release(ch) {
                break
            }
        }

        // 跳出 for range 循环， 意味着 从chan中取得一个 nil，或者 wp.mustStop 被设为了true，这是主动停止的方法。
        wp.lock.Lock()
        wp.workersCount--
        wp.lock.Unlock()
    }

for range 不断从 chan net.Conn 中获取连接。大家是否还记得 在
`func (wp *workerPool) Serve(c net.Conn) bool`
函数中，一个重要操作就是把 accept 到的connection，放入 channel.\
最后，需要把当前的 workerChan 释放回 workerPool 的 ready 中。

break-06
--------

    func (wp *workerPool) release(ch *workerChan) bool {
        ch.lastUseTime = CoarseTimeNow()
        wp.lock.Lock()
        if wp.mustStop {
            wp.lock.Unlock()
            return false
        }
        wp.ready = append(wp.ready, ch)
        wp.lock.Unlock()
        return true
    }

释放操作中，注意到 修改了 ch.lastUseTime ， 还记得 clean 操作吗？在 g1
协程中运行着呢。\
所以最后的运行状态是：

goroutine status:

1.  main0: wp.Start()

2.  g1: for loop to clean idle workerChan

3.  g2: wp.workerFunc(ch) blocks for handling connection

4.  g3: ....

5.  g4: ....

按需增长 goroutine 数量，但是也有一个最大值,
所以并行度是可控的。当请求密集时，一个 worker goroutine
可能会串行处理多个 connection。\
wokerChan 在 Pool 中被复用，对GC的压力会减小很多。

而对比原生的 net/http 包，并行度不可控（可能不确定，runtime 会有控制?
），goroutine 不可被复用，体现在一个请求一个goroutine,
用完就销毁了，对机器压力更大。

