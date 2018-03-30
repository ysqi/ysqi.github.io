
---
date: 2016-12-31T11:34:42+08:00
title: "Go程序的性能调试问题-内存篇"
description: ""
disqus_identifier: 1485833682589371341
slug: "Go-cheng-xu-de-xing-neng-diao-shi-wen-ti----nei-cun-pian"
source: "https://segmentfault.com/a/1190000000670041"
tags: 
- 性能分析 
- 性能调优 
- profiler 
- golang 
topics:
- 编程语言与开发
---

标签（空格分隔）： Go Memory Profiler 性能调试 性能分析

------------------------------------------------------------------------

> 注：该文作者是 [Dmitry
> Vyukov](https://software.intel.com/en-us/user/347692)，原文地址
> [Debugging performance issues in Go
> programs](https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs)

> 这个是原文中的 Memory Profiler 段落

内存分析器显示了函数分配堆内存的情况。你可以以 CPU profile
相似的方式收集：使用 [go test
--memprofile](http://golang.org/cmd/go/#hdr-Description_of_testing_flags)，通过
[http://myserver:6060:/debug/pprof/heap](http://myserver:6060:/debug/pprof/heap)
使用 [net/http/pprof](http://golang.org/pkg/net/http/pprof) 或是通过调用
[runtime/pprof.WriteHeapProfile](http://golang.org/pkg/runtime/pprof/#WriteHeapProfile)。

你仅可以显示在概要文件收集的时间分配的内存（默认，pprof 的
--inuse\_space 标志），或是从程序启动起的所有分配（pprof 的
--alloc\_space 标志）。前者对对于 net/http/pprof
的现场应用的概要文件收集非常有用，后者对程序结束的时候的概要文件收集非常有用（否则你将看到空荡荡的概要文件）。

> 注意：内存分析器很简单，也就是说，它收集的信息仅仅是关于内存分配的一些子集。概率抽样对象与它的大小成正比，你可以使用
> `go test --memprofilerate`
> 标志改变抽样比率，或者是在程序启动的时候设置 `runtime.MemProfileRate`
> 变量。比率 1
> 将导致收集所有分配的信息。但是它可能导致执行很慢，默认的采样率是每
> 512kb 的内存分配 1个样本。

你也可以显示分配的字节数，或者是分配的对象数量（`--inuse/alloc_space` 和
`--inuse/alloc_objects`
标志）。分析器在分析时更倾向于大样本对象。但是更重要的是要明白大对象影响内存消耗和
GC 时间，然而大量微小的分配影响执行速度（同样是某种程度的 GC
时间），所以两个都观察可能是非常有用的。

对象可以是持久的或是瞬态的。如果在程序开始的时候，你有一些大的持久化对象分配，它们将最有可能被分析器采样（因为它们足够大）。这样的对象会影响内存的消耗和
GC
时间，但是它们不影响正常的执行速度（没有内存管理操作发生在它们身上）。换句话说，如果你有大量的生命周期非常短暂的对象，在概要文件中，它们几乎可以代表（如果你使用默认的
--inuse\_space
模式），但它们很明显的会影响执行速度。因为它们在不断的分配和释放。因此，再一次声明，观察两种类型的对象是非常有用的。

因此，通常如果你想降低内存消耗，在正常的程序操作期间，你需要查看
`--inuse_space` 概要文件收集。如果你想提升执行速度，查看
`--alloc_objects` 概要文件收集，在重要的运行时间或程序结束之后。

这有一些标志控制报告的粒度。`--functions` 使得 `pprof`
报告在函数级别（默认）。`--lines` 使得 `pprof`
报告在源码的行级别。这是非常有用的，如果热函数在不同的行。这里也有
`--addresses` 和 `--files` 各自对应准确的指令地址和文件级别。

对于内存概要文件来说，这是非常有用的选项 --
你可以在浏览器中查看它（提供这个功能需要你 imported
net/http/pprof）。如果你打开
<http://myserver:6060/debug/pprof/heap?debug=1>，你必须看到堆类似：

    heap profile: 4: 266528 [123: 11284472] @ heap/1048576
    1: 262144 [4: 376832] @ 0x28d9f 0x2a201 0x2a28a 0x2624d 0x26188 0x94ca3 0x94a0b 0x17add6 0x17ae9f 0x1069d3 0xfe911 0xf0a3e 0xf0d22 0x21a70
    #    0x2a201    cnew+0xc1    runtime/malloc.goc:718
    #    0x2a28a    runtime.cnewarray+0x3a            runtime/malloc.goc:731
    #    0x2624d    makeslice1+0x4d                runtime/slice.c:57
    #    0x26188    runtime.makeslice+0x98            runtime/slice.c:38
    #    0x94ca3    bytes.makeSlice+0x63            bytes/buffer.go:191
    #    0x94a0b    bytes.(*Buffer).ReadFrom+0xcb        bytes/buffer.go:163
    #    0x17add6    io/ioutil.readAll+0x156            io/ioutil/ioutil.go:32
    #    0x17ae9f    io/ioutil.ReadAll+0x3f            io/ioutil/ioutil.go:41
    #    0x1069d3    godoc/vfs.ReadFile+0x133            godoc/vfs/vfs.go:44
    #    0xfe911    godoc.func·023+0x471            godoc/meta.go:80
    #    0xf0a3e    godoc.(*Corpus).updateMetadata+0x9e        godoc/meta.go:101
    #    0xf0d22    godoc.(*Corpus).refreshMetadataLoop+0x42    godoc/meta.go:141
    2: 4096 [2: 4096] @ 0x28d9f 0x29059 0x1d252 0x1d450 0x106993 0xf1225 0xe1489 0xfbcad 0x21a70
    #    0x1d252    newdefer+0x112                runtime/panic.c:49
    #    0x1d450    runtime.deferproc+0x10            runtime/panic.c:132
    #    0x106993    godoc/vfs.ReadFile+0xf3            godoc/vfs/vfs.go:43
    #    0xf1225    godoc.(*Corpus).parseFile+0x75        godoc/parser.go:20
    #    0xe1489    godoc.(*treeBuilder).newDirTree+0x8e9    godoc/dirtrees.go:108
    #    0xfbcad    godoc.func·002+0x15d            godoc/dirtrees.go:100

在每个入口开始的数字 ("1: 262144 \[4: 376832\]")
代表当前存活对象的数量，存活对象已经占用的内存，分配的总的数量和所有分配已经占用的内存。

优化通常特定于应用程序，但这里有一些常见的建议。

1.  对象合并成更大的对象。比如，使用 bytes.Buffer 代替 \*bytes.Buffer
    结构（后面你可以通过调用 bytes.Buffer.Grow 预先分配 buffer
    ）。这将降低内存的分配数量（更快），同时降低垃圾回收器的压力（更快的垃圾回收）。
2.  局部变量逃离了它们声明的范围，提升到堆分配。编译器通常不能证明几个变量有相同的寿命，因此它分别分配每个这样的变量。因此你可以使用以上的建议处理局部变量，比如，把下面这个：

        for k, v := range m {
           k, v := k, v   // copy for capturing by the goroutine
           go func() {
               // use k and v
           }()
        }

    替代为：

        for k, v := range m {
           x := struct{ k, v string }{k, v}   // copy for capturing by the goroutine
           go func() {
               // use x.k and x.v
           }()
        }

    这会把两个内存分配变为一个内存分配。尽管如此，该优化会影响代码的可读性，所以请合理使用它。

3.  分配的一个特例就是 slice 数组预分配。如果你知道一个 slice
    的标准大小，你可以像下面这样预分配一个支持数组：

        type X struct {
            buf      []byte
            bufArray [16]byte // Buf usually does not grow beyond 16 bytes.
        }

        func MakeX() *X {
            x := &X{}
            // Preinitialize buf with the backing array.
            x.buf = x.bufArray[:0]
            return x
        }

4.  如果可能的话，使用更小的数据类型，比如，使用 int8 代替 int。
5.  对象不包含任何指针（注意： strings，slices， maps 和 chans
    包含隐含的指针），不会被垃圾收集器扫描。比如，1GB byte 的 slice
    事实上不会影响垃圾收集时间。因此如果你从已经使用的活跃的对象移除指针，肯定会影响垃圾收集时间。一些可能性：使用
    indices 代替指针，把对象分割成两部分，其中一部分不包含指针。
6.  使用 freelists 重新利用瞬时对象和分配数量。标准包包含
    [sync.Pool](http://tip.golang.org/pkg/sync/#Pool)
    类型，在垃圾收集之间的几次，允许重新使用相同的对象。尽管如此，要知道，任何手动内存管理方案，
    不正确的使用 sync.Pool 可能会导致 use-after-free（释放后使用的 bug）
    bugs。

你可以使用垃圾收集器跟踪（见下文）来得到一些内存问题更深刻的见解。

