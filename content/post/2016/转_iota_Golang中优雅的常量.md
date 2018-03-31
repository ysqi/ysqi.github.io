
---
date: 2016-12-31T11:34:45+08:00
title: "iota:Golang中优雅的常量"
description: ""
disqus_identifier: 1485833685269835869
slug: "iota:-Golang-zhong-you-ya-de-chang-liang"
source: "https://segmentfault.com/a/1190000000656284"
tags: 
- iota 
- golang 
categories:
- 编程语言与开发
---

> 注：该文作者是 [Katrina
> Owen](https://blog.splice.com/author/katrina/)，原文地址是 [iota:
> Elegant Constants in
> Golang](https://blog.splice.com/iota-elegant-constants-golang/)

有些概念有名字，并且有时候我们关注这些名字，甚至（特别）是在我们代码中。

    const (
        CCVisa            = "Visa"
        CCMasterCard      = "MasterCard"
        CCAmericanExpress = "American Express"
    )

在其他时候，我们仅仅关注能把一个东西与其他的做区分。有些时候，有些时候一件事没有本质上的意义。比如，我们在一个数据库表中存储产品，我们可能不想以
string
存储他们的分类。我们不关注这个分类是怎样命名的，此外，该名字在市场上一直在变化。

我们仅仅关注它们是怎么彼此区分的。

    const (
        CategoryBooks    = 0
        CategoryHealth   = 1
        CategoryClothing = 2
    )

使用 0, 1, 和 2 代替，我们也可以选择 17， 43， 和 61。这些值是任意的。

常量是重要的，但是它们很难推断，并且难以维护。在一些语言中像 Ruby
开发者通常只是避免它们。在
Go，常量有许多微妙之处。当用好了，可以使得代码非常优雅且易维护的。

自增长
------

在 golang 中，一个方便的习惯就是使用 `iota`
标示符，它简化了常量用于增长数字的定义，给以上相同的值以准确的分类。

    const (
        CategoryBooks = iota // 0
        CategoryHealth       // 1
        CategoryClothing     // 2
    )

自定义类型
----------

自增长常量经常包含一个自定义类型，允许你依靠编译器。

    type Stereotype int

    const (
        TypicalNoob Stereotype = iota // 0
        TypicalHipster                // 1
        TypicalUnixWizard             // 2
        TypicalStartupFounder         // 3
    )

如果一个函数以 `int` 作为它的参数而不是 `Stereotype`，如果你给它传递一个
`Stereotype`，它将在编译器期出现问题。

    func CountAllTheThings(i int) string {
                    return fmt.Sprintf("there are %d things", i)
    }

    func main() {
        n := TypicalHipster
        fmt.Println(CountAllTheThings(n))
    }

    // output:
    // cannot use TypicalHipster (type Stereotype) as type int in argument to CountAllTheThings

相反亦是成立的。给一个函数以 `Stereotype` 作为参数，你不能给它传递
`int`。

    func SoSayethThe(character Stereotype) string {
        var s string
        switch character {
        case TypicalNoob:
            s = "I'm a confused ninja rockstar."
        case TypicalHipster:
            s = "Everything was better we programmed uphill and barefoot in the snow on the SUTX 5918"
        case TypicalUnixWizard:
            s = "sudo grep awk sed %!#?!!1!"
        case TypicalStartupFounder:
            s = "exploit compelling convergence to syndicate geo-targeted solutions"
        }
        return s
    }

    func main() {
        i := 2
        fmt.Println(SoSayethThe(i))
    }

    // output:
    // cannot use i (type int) as type Stereotype in argument to SoSayethThe

这是一个戏剧性的转折，尽管如此。你可以传递一个数值常量，然后它能工作。

    func main() {
        fmt.Println(SoSayethThe(0))
    }

    // output:
    // I'm a confused ninja rockstar.

这是因为常量在 Go 中是弱类型直到它使用在一个严格的上下文环境中。

Skipping Values
---------------

设想你在处理消费者的音频输出。音频可能无论什么都没有任何输出，或者它可能是单声道，立体声，或是环绕立体声的。

这可能有些潜在的逻辑定义没有任何输出为 0，单声道为 1，立体声为
2，值是由通道的数量提供。

所以你给 Dolby 5.1 环绕立体声什么值。

一方面，它有6个通道输出，但是另一方面，仅仅 5 个通道是全带宽通道（因此
5.1 称号 - 其中 `.1` 表示的是低频效果通道）。

不管怎样，我们不想简单的增加到 3。

我们可以使用下划线跳过不想要的值。

    type AudioOutput int

    const (
        OutMute AudioOutput = iota // 0
        OutMono                    // 1
        OutStereo                  // 2
        _
        _
        OutSurround                // 5
    )

表达式
------

`iota` 可以做更多事情，而不仅仅是 increment。更精确地说，`iota` 总是用于
increment，但是它可以用于表达式，在常量中的存储结果值。

这里我们创建一个常量用于位掩码。

    type Allergen int

    const (
        IgEggs Allergen = 1 << iota // 1 << 0 which is 00000001
        IgChocolate                         // 1 << 1 which is 00000010
        IgNuts                              // 1 << 2 which is 00000100
        IgStrawberries                      // 1 << 3 which is 00001000
        IgShellfish                         // 1 << 4 which is 00010000
    )

这个工作是因为当你在一个 `const`
组中仅仅有一个标示符在一行的时候，它将使用增长的 `iota`
取得前面的表达式并且再运用它，。在 Go 语言的
[spec](http://golang.org/ref/spec#Iota) 中，
这就是所谓的隐性重复最后一个非空的表达式列表。

如果你对鸡蛋，巧克力和海鲜过敏，把这些 bits 翻转到 “on”
的位置（从左到右映射 bits）。然后你将得到一个 bit 值
`00010011`，它对应十进制的 19。

    fmt.Println(IgEggs | IgChocolate | IgShellfish)

    // output:
    // 19

这是在 [Effective
Go](https://golang.org/doc/effective_go.html#constants)
中一个非常好定义数量级的示例：

    type ByteSize float64

    const (
        _           = iota                   // ignore first value by assigning to blank identifier
        KB ByteSize = 1 << (10 * iota) // 1 << (10*1)
        MB                                   // 1 << (10*2)
        GB                                   // 1 << (10*3)
        TB                                   // 1 << (10*4)
        PB                                   // 1 << (10*5)
        EB                                   // 1 << (10*6)
        ZB                                   // 1 << (10*7)
        YB                                   // 1 << (10*8)
    )

今天我学习到了在 zettabyte 之后是 yottabyte。

但是等等，这有更多
------------------

当你在把两个常量定义在一行的时候会发生什么？\
Banana 的值是什么？ 2 还是 3？ Durian 的值又是？

    const (
        Apple, Banana = iota + 1, iota + 2
        Cherimoya, Durian
        Elderberry, Fig
    )

`iota` 在下一行增长，而不是立即取得它的引用。

    // Apple: 1
    // Banana: 2
    // Cherimoya: 2
    // Durian: 3
    // Elderberry: 3
    // Fig: 4

这搞砸了，因为现在你的常量有相同的值。

**因此，对的**

在 Go 中，关于常量有很多东西可以说，你应该在 golang 博客读读 Rob Pike
的[这篇文章](http://blog.golang.org/constants)。

