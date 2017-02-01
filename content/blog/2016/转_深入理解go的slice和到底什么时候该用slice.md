
---
date: 2016-12-31T11:33:25+08:00
title: "深入理解go的slice和到底什么时候该用slice"
description: ""
disqus_identifier: 1485833605190732470
slug: "shen-ru-li-jie-gode-slicehe-dao-de-shen-me-shi-hou-gai-yong-slice"
source: "https://segmentfault.com/a/1190000005812839"
tags: 
- golang 
topics:
- 编程语言与开发
---

前言
----

用过go语言的亲们都知道，slice（中文翻译为切片）在编程中经常用到，它代表变长的序列，序列中每个元素都有相同的类型，类似一个动态数组，利用append可以实现动态增长，利用slice的特性可以很容易的切割slice，它们是怎么实现这些特性的呢？现在我们来探究一下这些特性的本质是什么。

先了解一下slice的特性
---------------------

定义一个slice：

    s := []int{1,2,3,4,5}
    fmt.Println(s)  // [1 2 3 4 5]

一个slice类型一般写作\[\]T，其中T代表slice中元素的类型；slice的语法和数组很像，只是没有固定长度而已。

slice的扩容：

    s := []int{1,2,3,4,5}
    s = append(s, 6)
    fmt.Println(s)  // [1 2 3 4 5 6]

内置append函数在现有数组的长度 &lt; 1024 时 cap
增长是翻倍的，再往上的增长率则是 1.25，至于为何后面会说。

slice的切割：

    s := []int{1,2,3,4,5,6}
    s1 := s[0:2]
    fmt.Println(s1)  // [1 2]
    s2 := s[4:]
    fmt.Println(s2)  // [5 6]
    s3 := s[:4]
    fmt.Println(s3)  // [1 2 3 4]

slice作为函数参数：

    package main

    import "fmt"

    func main() {

        slice_1 := []int{1, 2, 3, 4, 5}
        fmt.Printf("main-->data:\t%#v\n", slice_1)
        fmt.Printf("main-->len:\t%#v\n", len(slice_1))
        fmt.Printf("main-->cap:\t%#v\n", cap(slice_1))
        test1(slice_1)
        fmt.Printf("main-->data:\t%#v\n", slice_1)

        test2(&slice_1)
        fmt.Printf("main-->data:\t%#v\n", slice_1)

    }

    func test1(slice_2 []int) {
        slice_2[1] = 6666               // 函数外的slice确实有被修改
        slice_2 = append(slice_2, 8888) // 函数外的不变
        fmt.Printf("test1-->data:\t%#v\n", slice_2)
        fmt.Printf("test1-->len:\t%#v\n", len(slice_2))
        fmt.Printf("test1-->cap:\t%#v\n", cap(slice_2))
    }

    func test2(slice_2 *[]int) { // 这样才能修改函数外的slice
        *slice_2 = append(*slice_2, 6666)
    }

结果：

    main-->data:    []int{1, 2, 3, 4, 5}
    main-->len: 5
    main-->cap: 5
    test1-->data:   []int{1, 6666, 3, 4, 5, 8888}
    test1-->len:    6
    test1-->cap:    12
    main-->data:    []int{1, 6666, 3, 4, 5}
    main-->data:    []int{1, 6666, 3, 4, 5, 6666}

**这里要注意注释的地方，为何slice作为值传递参数，函数外的slice也被更改了？为何在函数内append不能改变函数外的slice？要回答这些问题就得了解slice内部结构，详细请看下面.**

slice的内部结构
---------------

其实slice在Go的运行时库中就是一个C语言动态数组的实现，在\$GOROOT/src/pkg/runtime/runtime.h中可以看到它的定义：

    struct    Slice
        {    // must not move anything
            byte*    array;        // actual data
            uintgo    len;        // number of elements
            uintgo    cap;        // allocated number of elements
        };

这个结构有3个字段，第一个字段表示array的指针，就是真实数据的指针（这个一定要注意），所以才经常说slice是数组的引用，第二个是表示slice的长度，第三个是表示slice的容量，注意：***len和cap都不是指针***。

现在就可以解释前面的例子slice作为函数参数提出的问题：

函数外的slice叫slice\_1，函数的参数叫slice\_2，当函数传递slice\_1的时候，其实传入的确实是slice\_1参数的复制，所以slice\_2复制了slise\_1，但要注意的是slice\_2里存储的数组的指针，所以当在函数内更改数组内容时，函数外的slice\_1的内容也改变了。在函数内用append时，append会自动以倍增的方式扩展slice\_2的容量，但是扩展也仅仅是函数内slice\_2的长度和容量，slice\_1的长度和容量是没变的，所以在函数外打印时看起来就是没变。

append的运作机制
----------------

在对slice进行append等操作时，可能会造成slice的自动扩容。其扩容时的大小增长规则是：

-   如果新的slice大小是当前大小2倍以上，则大小增长为新大小

-   否则循环以下操作：如果当前slice大小小于1024，按每次2倍增长，否则每次按当前大小1/4增长。直到增长的大小超过或等于新大小。

-   append的实现只是简单的在内存中将***旧slice***复制给***新slice***

至于为何会这样，你要看一下golang的源码[slice](https://github.com/golang/go/blob/master/src/runtime/slice.go)就知道了：

    newcap := old.cap
    if newcap+newcap < cap {
        newcap = cap
    } else {
        for {
            if old.len < 1024 {
                newcap += newcap
            } else {
                newcap += newcap / 4
            }
            if newcap >= cap {
                break
            }
        }
    }

为何不用动态链表实现slice？
---------------------------

-   首先拷贝一断连续的内存是很快的，假如不想发生拷贝，也就是用动态链表，那你就没有连续内存。此时随机访问开销会是：链表
    O(N), 2倍增长块链
    O(LogN),二级表一个常数很大的O(1)。问题不仅是算法上开销，还有内存位置分散而对缓存高度不友好，这些问题i在连续内存方案里都是不存在的。除非你的应用是狂append然后只顺序读一次，否则优化写而牺牲读都完全不
    make sense.
    而就算你的应用是严格顺序读，缓存命中率也通常会让你的综合效率比拷贝换连续内存低。

-   对小 slice 来说，连续 append 的开销更多的不是在 memmove,
    而是在分配一块新空间的 memory allocator 和之后的 gc
    压力（这方面对链表更是不利）。所以，当你能大致知道所需的最大空间（在大部分时候都是的）时，在make的时候预留相应的
    cap
    就好。如果所需的最大空间很大而每次使用的空间量分布不确定，那你就要在浪费内存和耗
    CPU 在 allocator + gc 上做权衡。

-   Go 在 append 和 copy
    方面的开销是可预知+可控的，应用上简单的调优有很好的效果。这个世界上没有免费的动态增长内存，各种实现方案都有设计权衡。

什么时候该用slice？
-------------------

在go语言中slice是很灵活的，大部分情况都能表现的很好，但也有特殊情况。\
当程序要求slice的容量超大并且需要频繁的更改slice的内容时，就不应该用slice，改用[list](https://golang.org/pkg/container/list/)更合适。

