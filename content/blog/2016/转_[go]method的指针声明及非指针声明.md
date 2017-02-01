
---
date: 2016-12-31T11:34:17+08:00
title: "[go]method的指针声明及非指针声明"
description: ""
disqus_identifier: 1485833657600858770
slug: "[go]methodde-zhi-zhen-sheng-ming-ji-fei-zhi-zhen-sheng-ming"
source: "https://segmentfault.com/a/1190000003772144"
tags: 
- method 
- golang 
topics:
- 编程语言与开发
---

刚入手golang，大概弄清楚了method在go里的概念。\
以下大部分为搬运过程重写代码。

------------------------------------------------------------------------

method可以为一个type添加(声明)一个方法，例如：

    type Cat struct {
    }

    func (c Cat) Hwo() {
        fmt.Println("Miah!")
    }

即对Cat类型(结构体)添加Hwo方法，使其在叫唤的时候可以发出声音。

这种添加方法的代码表现比java好很多(extend)，也比python好(需重新声明一个Class)。

------------------------------------------------------------------------

按官方的spec可以对应到如上的例子的两种声明：

    func (c Cat) Hwo()
    func (c *Cat) Hwo()

两种有什么区别呢？

    package main

    import (
        "fmt"
    )

    type Cat struct {
        age int
    }

    func (c Cat) AddAge() {
        fmt.Println("add age!")
        fmt.Println(c.age + 10)
        c.age += 1
    }

    func (c *Cat) AddOneAge() {
        c.age += 1
        fmt.Println("add one age!")
    }

    func main() {
        cat := &Cat{1}
        fmt.Println(cat)
        cat.AddAge()
        fmt.Println(cat)
        cat.AddOneAge()
        fmt.Println(cat)
    }

结果：

    &{1}
    add age!
    11
    &{1}
    add one age!
    &{2}

修改cat声明方式为

    cat := Cat{1}

结果：

    {1}
    add age!
    11
    {1}
    add one age!
    {2}

发生了什么？\
1.cat变量是一个指针，可以用reflect.Typeof(cat)看出来；\
2.(c
Cat)添加的方法AddAge()被执行了，能获取cat的值，但是未改变cat指针指向的内存区块的值；\
3.(c
\*Cat)添加的方法AddOneAge()被执行了，改变cat指针指向的内存区块的值；\
4.重新定义catb :=
Cat{1}，似乎catb非指针，但是还是一样的结果(除了变量部分)。

如何解读？\
1.无论是将一个变量声明为指针还是非指针，go在method上对待它们的态度都是一致的；\
2.声明method时，传入(c \*Cat)的声明方式才能修改new出来的对象(cat
:=)，因为method的处理对象是一个Cat类型的指针。\
3.在声明变量时，建议声明为指针对象cat := &Cat{1}，这样做有好处

> a.传递指针有卓有成效的，特别是传递大型变量时\
> b.大部分科学的代码是传递指针，为了代码的一致性最好都用指针

详见Reference.3

------------------------------------------------------------------------

题外话，这里有个例子可以阐明一个状况\
对于interface来说，识别method时会辨别(c Cat)及(c \*Cat)的区别：\
<http://play.golang.org/p/-g44WHg_uT>

Reference
---------

1.<http://nathanleclaire.com/blog/2014/08/09/dont-get-bitten-by-pointer-vs-non-pointer-method-receivers-in-golang/>\
2.<http://golang.org/ref/spec#Method_declarations>\
3.<http://golang.org/doc/faq#methods_on_values_or_pointers>

