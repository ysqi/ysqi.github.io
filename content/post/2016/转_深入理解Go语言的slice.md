
---
date: 2016-12-31T11:33:20+08:00
title: "深入理解Go语言的slice"
description: ""
disqus_identifier: 1485833600460760382
slug: "shen-ru-li-jie-Goyu-yan-de-slice"
source: "https://segmentfault.com/a/1190000006056800"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

先看这段代码，结果是`[0 2 3]`，很多人都能答对。

    func modify(s []int) {
        s[0] = 0
    }
    func main() {
        s := []int{1, 2, 3}
        modify(s)
        fmt.Println(s)
    }

然后稍微改动一下，再猜一下结果

    func pop(s []int) {
        s = s[:len(s)-1]
    }
    func main() {
        s := []int{1, 2, 3}
        pop(s)
        fmt.Println(s)
    }

如果认为输出`[1 2]`的话那么你错了，结果是`[1 2 3]`，你可能会觉得很奇怪，slice是引用语义这个在第一个例子中已经证明了，为什么第二个例子中又不是这样呢。

我们对中间过程加一些输出，再来看看

    func pop(s []int) {
        fmt.Printf("[pop] s addr:%p\n", &s)
        s = s[:len(s)-1]
        fmt.Println("[pop] s value:", s)
    }
    func main() {
        s := []int{1, 2, 3}
        fmt.Printf("[main] s addr:%p\n", &s)
        pop(s)
        fmt.Println("[main] s value:", s)
    }

运行上面代码输出如下

    [main] s addr:0xc082004640
    [pop] s addr:0xc0820046c0
    [pop] s value: [1 2]
    [main] s value: [1 2 3]

看到上面的结果，可以知道`pop()`中的`s`并不是引用，而是一个副本，虽然在`pop()`内部修改成功，但并没有影响到`main()`中的`s`。但第一个例子却修改成功了，这又是为什么。

下面来看下slice的实现，就能很清楚的了解原因了。\
slice是由长度固定的数组实现的。当使用内建函数`append()`向slice添加元素时，如果超过底层的数组长度则会重新分配空间（与C++的vector类似）。\
可以把slice认为是下面这样的一个结构体（先不考虑slice的容量）。`Lenght`表示slice的长度，\`ZerothElement表示底层数组的头指针

    type sliceHeader struct {
        Length        int
        ZerothElement *byte
    }

参照这个结构体的定义和下面的说明，就能很清楚地了解开始的两个例子了

那当我们需要将slice做为函数参数传入，并且函数会修改slice时，怎么办呢。这里说三种方法。\
1.将slice指针做为参数，而不是slice

    func modify(s *[]int) {
        // do something
    }

2.把函数内被修改后的slice做为返回值，将函数返回值赋值给原始slice

    func modify(s []int) []int {
        // do something
        return s
    }
    func main() {
        s := []int{1, 2, 3}
        s = modify(s)
    }

3.将函数做为slice指针的方法

    type slice []int

    func (s *slice) modify() {
        // do something
    }

