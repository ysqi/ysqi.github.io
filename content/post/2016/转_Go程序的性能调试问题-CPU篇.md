
---
date: 2016-12-31T11:34:53+08:00
title: "Go程序的性能调试问题-CPU篇"
description: ""
disqus_identifier: 1485833693137141407
slug: "Go-cheng-xu-de-xing-neng-diao-shi-wen-ti----CPU-pian"
source: "https://segmentfault.com/a/1190000000501635"
tags: 
- pprof 
- performance 
- debug 
- golang 
topics:
- 编程语言与开发
---

> 注：本文的原文 [Debugging performance issues in Go
> programs](https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs)
> 由 [Dmitry Vyukov](https://software.intel.com/en-us/user/347692) 在
> 05/10/2014 - 07:06 编写\
> 注：原文太长，大家要看全部的请看原文，其他的部分，后续慢慢翻译。

让我们假设你想提升你的GO程序的性能。这里有一些工具可以帮助你完成这个任务。这些工具能帮助你定位多种类型的热点（CPU，IO，内存），你为了能够显著提升程序性能，你必须专注于热点发生的地方。尽管如此，另外一个结果是可能的--这些工具能帮助你确定程序中明显的性能缺陷。例如，当你在每个程序启动的时候，你可以在每次查询的之前准备一个
SQL 语句。另外一个例子是如果一个 `O(N^2)`
算法在某种程度上陷入了一个明显存在并且期望的 `O(N)`
情况。为了确定这样的情况，你需要完整性的检查你在 profiles
中看到的。例如第一种情况下大量的时间花费在 SQL
语句的准备上，这已经超越了告警线了。

同样重要的是要理解性能影响的各种边界因素。例如，一个程序通过 100 Mbps
的带宽连接通讯，并且它已经使用超过 90
Mbps，这里就没有什么可以对这程序做的来提升它的性能了。这些类似的边界因素包括
磁盘 IO，内存消耗和计算任务。

考虑到这一点，我们可以查看这些可用的工具。

> 注意：这些工具可能相互干扰，例如，精确的内存分析可能影响CPU分析。goroutine
> 阻塞分析可能影响调度追踪等等，隔离地使用这些工具以便得到更加精确的信息。

> 注意：这里所有的描述都是基于 Go1.3 版本的

CPU 分析器
----------

Go runtime 包含了内建的 CPU 分析器，这显示了函数消耗的 CPU
时间百分比，这里你有3种方式访问它：

1.  最简单的一个方式是使用 `go test` 命令的
    [-cpuprofile](http://golang.org/cmd/go/#hdr-Description_of_testing_flags)
    标记。例如，如下命令：\
    `$ go test -run=none -bench=ClientServerParallel4 -cpuprofile=cprof net/http`\
    将配置给出的基准和 CPU 的概要分析写入 `cprof` 文件。\
    然后：\
    `$ go tool pprof --text http.test cprof`\
    将打印一份热点函数列表。\
    这有几个可用的输出类型，最有用的几个为： --text，--web 和 --list
    。运行 `go tool pprof` 来得到最完整的列表。\
    这个选项最明显的特点就是它只适用于测试。
2.  [net/http/pprof](http://golang.org/pkg/net/http/pprof)
    包。这是网络服务的理想解决方案。你仅仅只需要 `import net/http/pprof`
    并且用如下命令收集概要文件：

<!-- -->

    $ go tool pprof --text mybin http://myserver:6060:/debug/pprof/profile

1.  手工配置收集。你需要 import
    [runtime/pprof](https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs)
    并且 把如下代码加入 main 函数中：

<!-- -->

    if *flagCpuprofile != "" {
        f, err := os.Create(*flagCpuprofile)
        if err != nil {
            log.Fatal(err)
        }
        pprof.StartCPUProfile(f)
        defer pprof.StopCPUProfile()
    }

这份概要文件会被写入指定的文件中，想象它和第一个选项一样的命令行方式。

这里是一个使用 `--web` 选项产生的直观的概要文件示例：\

你可以使用 `-list=funcname` 选项来研究单个函数，下面的 概要文件显示了在
append 函数中所花费的时间：

    .      .   93: func (bp *buffer) WriteRune(r rune) error {
    .      .   94:     if r < utf8.RuneSelf {
    5      5   95:         *bp = append(*bp, byte(r))
    .      .   96:         return nil
    .      .   97:     }
    .      .   98: 
    .      .   99:     b := *bp
    .      .  100:     n := len(b)
    .      .  101:     for n+utf8.UTFMax > cap(b) {
    .      .  102:         b = append(b, 0)
    .      .  103:     }
    .      .  104:     w := utf8.EncodeRune(b[n:n+utf8.UTFMax], r)
    .      .  105:     *bp = b[:n+w]
    .      .  106:     return nil
    .      .  107: }

你可以在[这里](http://google-perftools.googlecode.com/svn/trunk/doc/cpuprofile.html)找到
`pprof` 工具的详细信息和描述的数字图。

当它不能解锁堆栈的时候，这里有关于分析器使用的3条特别条目：GC, System
and ExternalCode。GC
表示内存垃圾回收的时间花费，见下面的内存分析器和垃圾收集器跟踪优化建议部分。System
表示 goroutine 调度花费的时间，堆栈管理代码以及其他辅助的 runtime
代码。ExternalCode 表示本地动态库花费的时间。

这里有些提示关于如何解释说明你在 profile 中看到的信息。

如果很多时间花费在 `runtime.mallocgc`
函数，程序可能在小内存分配上面花费过多。 profile
会告诉你分配从哪里来，看内存分析器建议如何优化的部分。

如果很多时间花费在 `channel` 操作部分，`sync.Mutex` 代码和其他
`synchronization primitives`或者是系统组件，程序可能在承受资源竞争。考虑重构程序，以消除对共享资源的频繁访问。常见的技术包括分片、分区，本地缓冲/批量处理和写时拷贝技术。

如果很多时间花费在 `syscall.Read/Write`
上面，程序可能在小读写上面花费代价太大。Bufio 包装 `os.File` 或是
`net.Conn` 能对这种情况有帮助。

如果很多时间花费在 GC 组件上面，程序不是分配了太多的临时对象就是 heap
size
设置过小导致垃圾回收频繁发生。请看垃圾收集器跟踪和内存分析器优化建议部分。

> 注意： `CPU profiler` 目前[不能在 darwin
> 上工作](https://code.google.com/p/go/issues/detail?id=6047)\
> 注意： 在 windows 服务器上你需要安装 Cygwin, Perl and Graphviz 来生成
> svg/web 格式的概要文件\
> 注意： 在 Linux 上你可以尝试 [perf system
> profiler](https://perf.wiki.kernel.org/index.php/Tutorial)，它不能解锁
> GO 的堆栈，但是它能分析并且解锁 `cgo/SWIG`
> 代码和内核。所以它在定位分析本地/内核性能瓶颈上非常有用。

