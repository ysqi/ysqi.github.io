
---
date: 2016-12-31T11:34:28+08:00
title: "不是语言之争---GovsErlang"
description: ""
disqus_identifier: 1485833668251135743
slug: "bu-shi-yu-yan-zhi-zheng----Go-vs-Erlang"
source: "https://segmentfault.com/a/1190000002894381"
tags: 
- erlang 
- golang 
categories:
- 编程语言与开发
---

因为 [云巴](http://yunba.io)
系统对高并发、低延迟的需求，我们对各个语言、平台做了很多的调研比较工作。这自然就包括致力于开发高并发应用的
Go 和 Erlang。

并发
----

Go 对高并发的支持通过 goroutine 实现。goroutine 可以理解为轻量级的
线程（thread）。同一个 Go 应用创建的 goroutine 共享地址空间。

Erlang 的高并发通过轻量级
进程（process）实现，每一个进程都有独立的状态记录。

另外，使用 goroutine 要注意，goroutine
运行完毕后，占用的内存放回内存池备用，不会释放。

对于每一个任务都需要有独立状态的场景，Erlang 的 process 更有优势。

抢占式调度
----------

Erlang 的任务调度器有一个 reduction budget
的概念。进程的任何操作都会造成预算消耗，包括 函数调用、调用
BIF、进程堆垃圾回收、ETS
读写、发消息（目标邮箱堆积的消息越多，消耗越大）。Erlang 的 正则表达式库
也被做了修改以支持
reductions。所以如果进程在长时间执行正则表达式匹配，也一样会消耗
reductions，也会被抢占。

Go 之前的调度器只在 syscall
发生时调度，优化后可以在任何函数调用时调度。但是要注意，如果在 goroutine
里写一个死循环，Go 的调度器不能有效抢占，同一个调度器的 其他 goroutine
会被挂起。

垃圾回收
--------

像 Java 一样，Go 的垃圾回收是全局的，这意味着一旦垃圾回收被触发，所有的
goroutine 都会被暂停，造成一段时间的业务延迟。

Erlang 的垃圾回收是 进程
级别的，每一个进程都有自己独立的垃圾回收器，一个进程的垃圾回收被触发，不会造成其他进程被挂起。相对来说带来的业务延迟小。

错误处理
--------

Erlang 的每一个进程都有 进程 ID
（PID），同时也可以给进程注册名字，也就是说每一个进程都有独立的身份，可以有效的监控每一个进程的状态。进程异常退出时，可以捕捉到退出事件，并重启进程（参见
otp 的 supervisor/worker）。

Go 的 goroutine 没有身份识别，goroutine 的状态没办法监控。

动态反射
--------

Erlang 动态语言的特点，使它天然支持 REPL，另外 Erlang 支持 remote
shell，我们可以在 Erlang 运行时，连接到 remote shell
与任何一个进程交互。这些特性对一个需要长期运行的复杂系统的维护带来了极大的便利。开发阶段也能有一些便利。

Go 是静态语言，不支持 REPL。

静态编译
--------

Erlang 是动态语言，有所有动态语言的所有缺点：

> 1.  运行速度慢
> 2.  不能做早期错误检查，需要依赖全覆盖单元测试
> 3.  代码规模大了，给编写带来困扰

Erlang 现在也引入了
spec，对函数的参数返回值在编译时做类型检查，但是跟静态语言比起来效果差的很远。

不过正是因为是动态语言，Erlang
实现了运行时代码替换，这个特性对一个需要长时间运行的工业级产品，是一个非常重要的功能。

Go 是静态语言，运行速度快，编译时做严格的类型检查，可以避免很多隐患。

框架
----

Erlang 的 OTP 框架支持服务器端开发常见的几种模式（applications,
supervisors, wokers），方便代码的组织。

Go 暂时没看到类似的框架。

第三方库支持
------------

Go 是一个相对比较新的语言，虽说现在很多项目都开始支持
Go，但很多第三方库的成熟度暂时不如 Erlang。

总结
----

对于要求低延迟、高并发的后端服务，我们近期还是采用 Erlang 为主。但使用
Erlang 的过程中，Erlang
缺乏静态检查的手段，也是一个很麻烦的问题，目前的做法是要求大家都使用
IntelliJ IDEA 编写代码，可以通过 IDE 提前发现部分语言问题。

同时我们会持续关注 Go 的发展。

关于作者：\
weibo: [@Tiger\_张虎](http://weibo.com/zhanghusz)， 云巴 (yunba.io)
创始人，yunba.io 云后端服务。 JPush 创始人，原CTO。 Oracle VM
创始团队成员。\
原文：<http://zhang.hu/go-vs-erlang/>

