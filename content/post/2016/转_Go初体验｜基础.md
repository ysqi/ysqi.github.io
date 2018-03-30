
---
date: 2016-12-31T11:33:51+08:00
title: "Go初体验｜基础"
description: ""
disqus_identifier: 1485833631590802849
slug: "Gochu-ti-yan-｜ji-chu"
source: "https://segmentfault.com/a/1190000004852091"
tags: 
- golang 
topics:
- 编程语言与开发
---

字符串
======

GO的字符串有点不一样。它是UTF8字符的一个序列：当字符为一个ASCII码时为一个字节，其他字符则根据需要占用2-4个字节。

该做法的好处是：节省了内存和硬盘的存储空间，同时，不需要像其它语言一样对UTF8字符集的文本进行编码和解码。

GO通过双引号`"`和反引号`` ` ``来构建字符串。

双引号则会对文本进行解析，`` ` ``则不会。

单个字符可以通过`'`来创建，一个单一字符可以用一个单一的`rune`类型来表示。

字符串支持`切片`操作，但小心如果字符串包含非ASCII字符，切片则要小心。因为切片操作是使用对字符串的字节进行索引的。这时可以使用`range`来代替切片操作：对字符串的每一个字符进行操作。

字符串处理相关的包：

-   fmt: 打印字符串的格式

-   strings: 字符串处理函数

-   strconv(将字符串和其它类型数据进行转换)

值, 指针, 引用
==============

通常情况下，我们可以把一个GO变量当作他的值来使用。

其中有一些例外：通道、函数、方法、映射、切片都使用了`引用`，即保存指针的变量。

GO中的指针操作符是： & 和 \*

`&` 用于取地址\
`*` 用于解引用，即获取指针指向的值

    package main

    import (
        "fmt"
    )

    func swap1(x, y, p *int){
        if *x > *y {
            *x, *y = *y, *x
        }
        *p = *x * *y
    }

    func swap2(x, y int)(int, int, int){
        if x > y {
            x, y = y, x
        }
        return x, y, x*y
    }

    func main(){
        i := 9
        j := 5
        product := 0
        swap1(&i, &j, &product)
        fmt.Println(i, j, product)

        a := 63
        b := 64
        a, b, p := swap2(a, b)
        fmt.Println(a, b, p)
    }

数组
====

GO的数组是一个定长的序列，元素类型相同。通过`[]`来构建索引。

构建语法

    [length]Type
    [N]Type{v1,v2,vN}
    []Type{}

数组的长度是固定，不可修改的。可以通过`len()`来获得数组的长度，`cap()`获得数组的切片大小（不确定）。

数组也有切片操作，也可以通过`range`进行索引访问。

切片
====

GO的数组是值传递，而切片是引用传递，因此效率更高。

创建切片

    make([]Type, length, capacity)
    make([]Type, length)
    []Type{}
    []Type{v1, v2, vN}

内置函数`make`来创建切片，映射和通道。

当创建一个切片时，首先会创建一个隐藏的初始化为零值的数组，然后返回一个引用该隐藏数组的切片。一个切片的容量（capacity）为隐藏数组的长度。可以通过`append`来增加切片的容量。

未完待续

映射
====

参考

1.  [https://www.shiyanlou.com/courses/runnin...](https://www.shiyanlou.com/courses/running/127)



