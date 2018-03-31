
---
date: 2016-12-31T11:34:26+08:00
title: "go如何使用SIMD指令"
description: ""
disqus_identifier: 1485833666101097024
slug: "go-ru-he-shi-yong--SIMD-zhi-ling"
source: "https://segmentfault.com/a/1190000002982384"
tags: 
- lucene 
- golang 
categories:
- 编程语言与开发
---

Java SIMD Lucene Elasticsearch
------------------------------

我们首先来看一下 JAVA 如何使用 CPU 的 SIMD
指令。这是一个ru的哥们尝试在lucene里使用SIMD指令加速lucene的postings
list（也就是指定term对应的文档id列表）的解码：

[http://blog.griddynamics.com/2015/02/proposing-simd-codec-for-lucene.h...](http://blog.griddynamics.com/2015/02/proposing-simd-codec-for-lucene.html)\
[https://www.youtube.com/watch?v=2HQdbpgHfnQ&index=15&list=PLq-...](https://www.youtube.com/watch?v=2HQdbpgHfnQ&index=15&list=PLq-odUc2x7i-_qWWixXHZ6w-MxyLxEC7s)

最重要的结论就是 java
自身还不支持JIT（运行时产生的机器码）出SIMD指令。如果用 c/asm 编写 SIMD
的代码，在 java 里调用的话 JNI 本身的开销抵消了 SIMD
带来的好处。所以最终需要使

用一种更底层的方式访问 native 代码：

[http://stackoverflow.com/questions/24746776/what-does-a-jvm-have-to-do...](http://stackoverflow.com/questions/24746776/what-does-a-jvm-have-to-do-when-calling-a-native-method/24747484#24747484)

值得一提的是 elasticsearch 从 2.0 大幅加强了
aggregation，现在已经开始支持 pipeline 了。可以写出类似 select
sum(money) / sum(users\_count) from payment 之类的代码了。自然 SIMD
的优化也可以做到 aggregation 阶段里去。

[https://www.elastic.co/guide/en/elasticsearch/reference/master/search-...](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-pipeline-bucket-script-aggregation.html)

Go CGO
------

CGO 慢，显而易见。

<https://github.com/golang/go/blob/master/src/runtime/cgocall.go>

具体来说就是这几行

        /*
         * Announce we are entering a system call
         * so that the scheduler knows to create another
         * M to run goroutines while we are in the
         * foreign code.
         *
         * The call to asmcgocall is guaranteed not to
         * split the stack and does not allocate memory,
         * so it is safe to call while "in a system call", outside
         * the $GOMAXPROCS accounting.
         */
        entersyscall(0)
        errno := asmcgocall(fn, arg)
        exitsyscall(0)

每次调用 c 的函数都假设了这个函数是阻塞的。entersyscall
会保存当前协程的堆栈信息。所以Go的策略和Java一样，通过让JNI很慢，迫使用户把尽可能多的代码都写到Go里。

Go Plan9 Assembly
-----------------

Go有两个编译器，一个是gc（go
compiler），一个是gccgo（用的是gcc的后端）。gc编译器是把代码从go编译成plan
9的汇编。plan 9的汇编不是平台无关的，而是每个平台有一个版本，然后和这

个平台本身的汇编语法又有不同。\
首先我们可以来看一下 gc 编译器是不是会产生 SIMD 指令：

[https://github.com/golang/go/blob/master/src/cmd/compile/internal/amd6...](https://github.com/golang/go/blob/master/src/cmd/compile/internal/amd64/prog.go)

可以看到，在这个列表里是没有 ADDPD 这样的 SIMD 指令的。说明 gc
编译器目前还不支持把普通的加法编译成向量加法。用 Intel
的编译器，如果把代码协程 struct of array 的形式而不是

array of struct 形式的话，编译器可以自动做向量化优化。显然 gc
编译器还没有把这个做为一个优化方向。

[https://software.intel.com/sites/default/files/8c/a9/CompilerAutovecto...](https://software.intel.com/sites/default/files/8c/a9/CompilerAutovectorizationGuide.pdf)

虽然gc编译器不支持 SIMD，但是其 plan9 的 assembler 是支持在 amd64 的
SIMD 指令的。

[https://github.com/golang/go/blob/master/src/cmd/internal/obj/x86/asm6...](https://github.com/golang/go/blob/master/src/cmd/internal/obj/x86/asm6.go)

其中有 AADDPD （也就是 ADDPD）。而 Go 是支持在代码里混用 go 和 plan9
汇编的。所以 gonum 这个项目就写了一些 plan9 汇编来优化性能：

<https://github.com/gonum/internal/blob/master/asm/ddot_amd64.s>

简单做了一个benchmark：

    package main

    import "fmt"
    import "simd/asm"
    import "testing"

    func BenchmarkFunction(b *testing.B) {
        x := make([]float64, 10000)
        for i := 0; i < len(x); i++ {
            x[i] = float64(i)
        }
        y := make([]float64, 10000)
        for i := 0; i < len(y); i++ {
            y[i] = float64(i)
        }
        for i := 0; i < b.N; i++ {
            _ = asm.DdotUnitary(x, y)
        }
    }

    func main() {
        br := testing.Benchmark(BenchmarkFunction)
        fmt.Println(br)
    }

使用 SIMD 版本的点乘，速度为 4616 ns/op。使用非 SIMD 版本的点乘，速度为
12340 ns/op。目前 Go 并不支持 inline plan9
的汇编代码。也就是汇编写的函数每次调用都要付出一个函数call

的成本，也就是没法当成 SIMD intrinsics 那样来用。不过仍然比 java
强多了……

GCCGO
-----

Go还有另外一个编译器。它提供了另外一种Cgo的方式，extern。

<https://golang.org/doc/install/gccgo>

使用 extern 可以把任意的 c 的代码链接到 go 代码里来。至于 scheduler 和
garbage collector 这些就自己好自为之了。甚至类型互相转换的细节都还是
subject to change 的。可以把它理解

为去掉了安全保护的 cgo。

利用这条路也可以把 SIMD 指令链接到 go 代码里来使用：

[http://stackoverflow.com/questions/2951028/is-it-possible-to-include-i...](http://stackoverflow.com/questions/2951028/is-it-possible-to-include-inline-assembly-in-google-go-code)

使用 gccgo 可能还可以把这些 SIMD 调用在link时做inline：

<https://groups.google.com/forum/#!topic/golang-nuts/kGgkcOFCBtc>\
<https://groups.google.com/forum/#!topic/golang-nuts/TqMTWdYGKOk>

引用一段

    Answering specifically about gccgo.  Gccgo is of course just a 
    frontend to GCC.  GCC can not inline functions written in pure 
    assembly.  However, GCC provides CPU-specific builtin functions usable 
    in C/C++ for many things that people want to do (e.g., vector 
    instructions) and it also provides a sophisticated asm expression as a 
    C/C++ extension.  This means that you can write your assembly code in 
    extended C/C++ instead, and a function written that way can be 
    inlined.  It can even be inlined into Go code if you use LTO 
    (link-time optimization, see GCC's -flto options). 

总结
----

Go有三种调用native的代码的方式：

-   cgo
-   plan9 assembly
-   gccgo extern

相比Java的JNI来说，可选项更多。不远的将来 go 可以在 spark/lucene
这两个领域从速度上超过 Java。\
go 1.5 的编译器已经是用 go 写的。也许将来 go 的编译器可以和 Intel
的编译器一样，自动产生向量化的代码。

