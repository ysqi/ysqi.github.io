
---
date: 2016-12-31T11:32:57+08:00
title: "PHP混合Go协程并发"
description: ""
disqus_identifier: 1485833577627065711
slug: "PHP-hun-ge--Go-xie-cheng-bing-fa"
source: "https://segmentfault.com/a/1190000007299367"
tags: 
- php 
- golang 
- coroutine 
topics:
- 编程语言与开发
---

想法很简单。通过设置 `runtime.GOMAXPROCS(1)` 让 golang
的进程变成单线程执行的。类似python用gevent的效果。然后通过调度多个协程实现异步I/O并发。php作为一个子函数跑在go的进程内，php需要yield到其他协程时，通过回调到golang函数来实现。从php里调用go提供的子函数时，go保证保存php的当前上下文。当协程执行权让渡回来的时候，把原来的php上下文恢复。关键的代码在：

```Go
    // 保存当前协程上的php上下文
        oldServerCtx := engine.ServerContextGet()
        fmt.Println(oldServerCtx)
        defer engine.ServerContextSet(oldServerCtx)
        oldExecutorCtx := engine.ExecutorContextGet()
        fmt.Println(oldExecutorCtx)
        defer engine.ExecutorContextSet(oldExecutorCtx)
        oldCoreCtx := engine.CoreContextGet()
        fmt.Println(oldCoreCtx)
        defer engine.CoreContextSet(oldCoreCtx)

    // 放弃全局的锁，使得其他的协程可以开始执行php
        engineLock.Unlock()
        defer engineLock.Lock()
```

ServerContextGet
这几个函数是我加的，获得的是php的（EG/SG/PG）这三个全局context（参见：[http://www.cnblogs.com/chance...](http://www.cnblogs.com/chance1/archive/2009/12/28/1634515.html)）。修改过的`github.com/deuill/go-php`的源代码在：[https://github.com/taowen/go-...](https://github.com/taowen/go-php/commit/cfe707720bb4f3be60ae571fddfaa660194be9b5)

完整的php/go混合协程的demo：

```Go
    package main

    import (
        "fmt"
        "github.com/deuill/go-php/engine"
        "os"
        "runtime"
        "time"
        "sync"
    )

    type TestObj struct{}

    func newTestObj(args []interface{}) interface{} {
        return &TestObj{}
    }
    var engineLock *sync.Mutex

    func (self *TestObj) Hello() {
        oldServerCtx := engine.ServerContextGet()
        fmt.Println(oldServerCtx)
        defer engine.ServerContextSet(oldServerCtx)
        oldExecutorCtx := engine.ExecutorContextGet()
        fmt.Println(oldExecutorCtx)
        defer engine.ExecutorContextSet(oldExecutorCtx)
        oldCoreCtx := engine.CoreContextGet()
        fmt.Println(oldCoreCtx)
        defer engine.CoreContextSet(oldCoreCtx)
        engineLock.Unlock()
        defer engineLock.Lock()
        time.Sleep(time.Second)
        fmt.Println("sleep done")
    }

    func main() {
        runtime.GOMAXPROCS(1)
        theEngine, err := engine.New()
        engineLock = &sync.Mutex{}
        if err != nil {
            fmt.Println(err)
        }
        _, err = theEngine.Define("TestObj", newTestObj)
        wg := &sync.WaitGroup{}
        wg.Add(2)
        before := time.Now()
        fmt.Println("1")
        go func() {
            engineLock.Lock()
            defer engineLock.Unlock()
            context1, err := theEngine.NewContext()
            if err != nil {
                fmt.Println(err)
            }
            context1.Output = os.Stdout
            if err != nil {
                fmt.Println(err)
            }
            fmt.Println("1 enter")
            _, err = context1.Eval("$testObj = new TestObj(); $testObj->Hello();")
            fmt.Println("1 back")
            if err != nil {
                fmt.Println(err)
            }
            //theEngine.DestroyContext(context1)
            fmt.Println("1 done")
            wg.Done()
        }()
        fmt.Println("2")
        go func() {
            engineLock.Lock()
            defer engineLock.Unlock()
            context2, err := theEngine.NewContext()
            if err != nil {
                fmt.Println(err)
            }
            if err != nil {
                fmt.Println(err)
            }
            context2.Output = os.Stdout
            fmt.Println("2 enter")
            _, err = context2.Eval("$testObj = new TestObj(); $testObj->Hello();")
            fmt.Println("2 back")
            if err != nil {
                fmt.Println(err)
            }
            //theEngine.DestroyContext(context2)
            fmt.Println("2 done")
            wg.Done()
        }()
        wg.Wait()
        after := time.Now()
        fmt.Println(after.Sub(before))
    }
```

执行结果是

```
    1
    2
    2 enter
    {0x2cf2930 {<nil> <nil> <nil> 0 <nil> <nil> <nil> <nil> 0 0 0 [0 0 0 0 0] <nil> <nil> <nil> <nil> <nil> <nil> <nil> 0 0 <nil> 1000 [0 0 0 0]} \{\{<nil> <nil> 0 16 0x7f682e819780 0 [0 0 0 0 0 0 0] <nil>} 0 1 [0 0 0] <nil> <nil>} 0 0 0 [0 0 0 0 0 0] {0 0 0 0 0 0 0 0 0 0 0 {0 0} {0 0} {0 0} [0 0 0]} 0x2a00270 0x2a00f60 <nil> 8388608 0 1 [0 0 0] 0 {8 7 2 [0 0 0 0] 0 0x29f4520 0x29f4520 0x29f4470 0x29f4420 <nil> 1 0 0 [0 0 0 0 0]} <nil> {0 [0 0 0 0 0 0 0] <nil> <nil> <nil> <nil>} 0 [0 0 0 0 0 0 0]}
    {0x7ffd30bac588 {[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] 2 0 0 [0 0]} 0x7f682f01b928 {[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] 1 0 0 [0 0]} 0x7f682f01b948 [<nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>] 0x7f682f01ba60 0x7f682f01b960 0x7f682f167168 0x7f682f01ba88 {64 63 5 [0 0 0 0] 0 0x7f682f1972d8 0x7f682f1972d8 0x7f682f1993f8 0x7f682f1970c8 0x7f682e862d10 0 0 1 [0 0 0 0 0]} {8 0 0 [0 0 0 0] 0 <nil> <nil> <nil> 0x7f682f016a00 <nil> 0 0 1 [0 0 0 0 0]} 0x7ffd30bac590 22527 0 0 [0 0 0 0] 0x7f682f197640 0x29f4f80 0x29f4fd0 0x29f5070 <nil> 0x2cf2950 0x7f682f1989c0 14 0 1 [0 0 0] <nil> <nil> 0 1 [0 0 0 0 0 0] {8 0 0 [0 0 0 0] 1 <nil> <nil> <nil> 0x7f682f016a00 0x7f682e883140 0 0 1 [0 0 0 0 0]} {8 0 0 [0 0 0 0] 0 <nil> <nil> <nil> 0x7f682f016a00 0x7f682e8831d0 1 0 0 [0 0 0 0 0]} 0x7f682f167088 0 [0 0 0 0] <nil> <nil> {0 0 <nil>} {0 0 <nil> <nil> 0 [0 0 0 0 0 0 0]} {0 0 <nil> <nil> 0 [0 0 0 0 0 0 0]} 0 [0 0 0 0] <nil> 0 0 0x29fb2e0 <nil> <nil> {0x7f682f187030 2 1024 -1 [0 0 0 0]} <nil> <nil> <nil> [{0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8} {0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8} {0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8}] 0x7f682f167168 <nil> {0 [0 0 0 0] <nil> 0 [0 0 0 0] 0 0 [0 0 0 0] <nil> 0 [0 0 0 0] <nil>} 1 [0 0 0 0 0 0 0] <nil> 0x7f682f01bde8 895 [0 0 0 0 0 0] [<nil> <nil> <nil> <nil>]}
    {1 [0 0 0 0 0 0 0] 0 0 0 [0 0 0 0 0 0] <nil> 0x29ff9a0 17 134217728 -1 0 0 0 1 [0 0 0 0] 1024 0 0 1 [0 0 0 0 0] 0x2a00870 <nil> 0x2a010a0 0x7f682ecc58b0 <nil> 0x7f682ecc5c23 <nil> <nil> <nil> 2097152 <nil> <nil> 0x2a00180 0x2a00230 <nil> <nil> <nil> {0x7f682ec91aa8 0x7f682ec91aa8} 0x2a00910 {0 0 0 [0 0 0 0] 0 <nil> <nil> <nil> <nil> <nil> 0 0 0 [0 0 0 0 0]} 0 0 0 [0 0 0] {0x2b6dc10 0x2b6dc10 1 8 <nil> 1 [0 0 0 0 0 0 0] <nil>} [0x7f682f197330 0x7f682f197040 0x7f682f197410 <nil> <nil> 0x7f682f1974f0] 0 1 1 [0 0 0 0 0] 0x7f682ec9544b 0x7f682ec9544b 0 0 [0 0 0 0 0 0] 0 [0 0 0 0 0 0 0 0] 1 1 1 1 1 0 1 [0] 0 [0 0 0 0] <nil> <nil> 0 [0 0 0 0] 0x2cf27c0 <nil> 0 0 [0 0 0 0 0 0] 64 1000 0 [0 0 0 0 0 0 0] 0x7f682ecc6270 300 0x2a009b0 1 [0 0 0 0 0 0 0] <nil> 0 [0 0 0 0 0 0 0]}
    1 enter
    {0x7f6818000aa0 {<nil> <nil> <nil> 0 <nil> <nil> <nil> <nil> 0 0 0 [0 0 0 0 0] <nil> <nil> <nil> <nil> <nil> <nil> <nil> 0 0 <nil> 1000 [0 0 0 0]} \{\{<nil> <nil> 0 16 0x7f682e819780 0 [0 0 0 0 0 0 0] <nil>} 0 1 [0 0 0] <nil> <nil>} 0 0 0 [0 0 0 0 0 0] {0 0 0 0 0 0 0 0 0 0 0 {0 0} {0 0} {0 0} [0 0 0]} 0x2a00270 0x2a00f60 <nil> 8388608 0 1 [0 0 0] 0 {8 7 2 [0 0 0 0] 0 0x29f4520 0x29f4520 0x29f4470 0x29f4420 <nil> 1 0 0 [0 0 0 0 0]} <nil> {0 [0 0 0 0 0 0 0] <nil> <nil> <nil> <nil>} 0 [0 0 0 0 0 0 0]}
    {0x7f682a4cccd8 {[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] 2 0 0 [0 0]} 0x7f682f01b928 {[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0] 1 0 0 [0 0]} 0x7f682f01b948 [<nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>] 0x7f682f01ba60 0x7f682f01b960 0x7f682802f110 0x7f682f01ba88 {64 63 5 [0 0 0 0] 0 0x7f682f197a00 0x7f682f197a00 0x7f682f198368 0x7f682f198fa0 0x7f682e862d10 0 0 1 [0 0 0 0 0]} {8 0 0 [0 0 0 0] 0 <nil> <nil> <nil> 0x7f682f016a00 <nil> 0 0 1 [0 0 0 0 0]} 0x7f682a4ccce0 22527 0 0 [0 0 0 0] 0x7f682f197d28 0x29f4f80 0x29f4fd0 0x29f5070 <nil> 0x2cf2950 0x7f682f1983e8 14 0 1 [0 0 0] <nil> <nil> 0 1 [0 0 0 0 0 0] {8 0 0 [0 0 0 0] 1 <nil> <nil> <nil> 0x7f682f016a00 0x7f682e883140 0 0 1 [0 0 0 0 0]} {8 0 0 [0 0 0 0] 0 <nil> <nil> <nil> 0x7f682f016a00 0x7f682e8831d0 1 0 0 [0 0 0 0 0]} 0x7f682802f030 0 [0 0 0 0] <nil> <nil> {0 0 <nil>} {0 0 <nil> <nil> 0 [0 0 0 0 0 0 0]} {0 0 <nil> <nil> 0 [0 0 0 0 0 0 0]} 0 [0 0 0 0] <nil> 0 0 0x29fb2e0 <nil> <nil> {0x7f682804efd8 2 1024 -1 [0 0 0 0]} <nil> <nil> <nil> [{0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8} {0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8} {0x7f682e915050 [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] [0 0 0 0 0 0 0 0] 0 0 149 8 8 8}] 0x7f682802f110 <nil> {0 [0 0 0 0] <nil> 0 [0 0 0 0] 0 0 [0 0 0 0] <nil> 0 [0 0 0 0] <nil>} 1 [0 0 0 0 0 0 0] <nil> 0x7f682f01bde8 895 [0 0 0 0 0 0] [<nil> <nil> <nil> <nil>]}
    {1 [0 0 0 0 0 0 0] 0 0 0 [0 0 0 0 0 0] <nil> 0x29ff9a0 17 134217728 -1 0 0 0 1 [0 0 0 0] 1024 0 0 1 [0 0 0 0 0] 0x2a00870 <nil> 0x2a010a0 0x7f682ecc58b0 <nil> 0x7f682ecc5c23 <nil> <nil> <nil> 2097152 <nil> <nil> 0x2a00180 0x2a00230 <nil> <nil> <nil> {0x7f682ec91aa8 0x7f682ec91aa8} 0x2a00910 {0 0 0 [0 0 0 0] 0 <nil> <nil> <nil> <nil> <nil> 0 0 0 [0 0 0 0 0]} 0 0 0 [0 0 0] {0x2b6dc10 0x2b6dc10 1 8 <nil> 1 [0 0 0 0 0 0 0] <nil>} [0x7f682f197a58 0x7f682f198ce0 0x7f682f197b38 <nil> <nil> 0x7f682f197c18] 0 1 1 [0 0 0 0 0] 0x7f682ec9544b 0x7f682ec9544b 0 0 [0 0 0 0 0 0] 0 [0 0 0 0 0 0 0 0] 1 1 1 1 1 0 1 [0] 0 [0 0 0 0] <nil> <nil> 0 [0 0 0 0] 0x2cf27c0 <nil> 0 0 [0 0 0 0 0 0] 64 1000 0 [0 0 0 0 0 0 0] 0x7f682ecc6270 300 0x2a009b0 1 [0 0 0 0 0 0 0] <nil> 0 [0 0 0 0 0 0 0]}
    sleep done
    1 back
    1 done
    sleep done
    2 back
    2 done
    1.00099211s
```

可以看到两个sleep 1s，最终只用了1.00099211s。说明协程是并发的。

一些性能指标。走http调用后端，在i7-6700k上，用ab -n 100 -c 4
可以跑出这样的结果

    Requests per second:    3183.70 [#/sec] (mean)
    Time per request:       1.256 [ms] (mean)
    Time per request:       0.314 [ms] (mean, across all concurrent requests)

如果不用http调用后端，直接php=&gt;go返回"hello"，则可以达到

    Requests per second:    10073.54 [#/sec] (mean)
    Time per request:       0.397 [ms] (mean)
    Time per request:       0.099 [ms] (mean, across all concurrent requests)

这些指标只说明了协程切换的成本。实际的收益取决于后端的http服务的延迟，如果耗时很长，通过协程并发则可以收益明显。

这个实验说明了可以用golang实现一个代替nginx+php-fpm的应用服务器。并且提供了一条从php向golang迁移的平滑迁移路径。在一个应用里混合PHP和Go两种语言。

并且可以通过提供golang函数给php调用的方式实现I/O的异步化。像libcurl这样的扩展自身是支持异步回调的，只是php是同步的所以只给php暴露了同步的execute。有了Golang之后，可以把execute变成对异步execute+callback的包装，从而实现基于协程的调度。

参考资料：

-   [https://wiki.php.net/internal...](https://wiki.php.net/internals/references)

-   [http://www.cunmou.com/phpbook...](http://www.cunmou.com/phpbook/preface.md)

-   [http://www.phpinternalsbook.c...](http://www.phpinternalsbook.com/index.html)

-   [http://www.php-internals.com/...](http://www.php-internals.com/book/)