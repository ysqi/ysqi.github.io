
---
date: 2016-12-31T11:33:15+08:00
title: "Go语言反射三定律"
description: ""
disqus_identifier: 1485833595143150280
slug: "Go-yu-yan-fan-she-san-ding-lv"
source: "https://segmentfault.com/a/1190000006190038"
tags: 
- 反射 
- reflect 
- golang 
categories:
- 编程语言与开发
---

简介
----

Reflection（反射）在计算机中表示
程序能够检查自身结构的能力，尤其是类型。它是元编程的一种形式，也是最容易让人迷惑的一部分。

本文中，我们将解释Go语言中反射的运作机制。每个编程语言的反射模型不大相同，很多语言索性就不支持反射（C、C++）。由于本文是介绍Go语言的，所以当我们谈到“反射”时，默认为是Go语言中的反射。

阅读建议
--------

本文中，我们将解释Go语言中反射的运作机制。每个编程语言的反射模型不大相同，很多语言索性就不支持反射（C、C++）。

由于本文是介绍Go语言的，所以当我们谈到“反射”时，默认为是Go语言中的反射。

虽然Go语言没有继承的概念，但为了便于理解，如果一个struct A 实现了
interface B的所有方法时，我们称之为“继承”。

类型和接口
----------

反射建立在类型系统之上，因此我们从类型基础知识说起。

Go是静态类型语言。每个变量都有且只有一个静态类型，在编译时就已经确定。比如
int、float32、\*MyType、\[\]byte。 如果我们做出如下声明：

    type MyInt int

    var i int
    var j MyInt

上面的代码中，变量 i 的类型是 int，j 的类型是 MyInt。 所以，尽管变量 i
和 j 具有共同的底层类型
int，但它们的静态类型并不一样。不经过类型转换直接相互赋值时，编译器会报错。

关于类型，一个重要的分类是
接口类型（interface），每个接口类型都代表固定的方法集合。一个接口变量就可以存储（或“指向”，接口变量类似于指针）任何类型的具体值，只要这个值实现了该接口类型的所有方法。一组广为人知的例子是
io.Reader 和 io.Writer， Reader 和 Writer 类型来源于 io包，声明如下：

    // Reader is the interface that wraps the basic Read method.
    type Reader interface {
        Read(p []byte) (n int, err error)
    }

    // Writer is the interface that wraps the basic Write method.
    type Writer interface {
        Write(p []byte) (n int, err error)
    }

任何实现了 Read（Write）方法的类型，我们都称之为继承了
io.Reader（io.Writer）接口。换句话说， 一个类型为 io.Reader 的变量
可以指向（接口变量类似于指针）任何类型的变量，只要这个类型实现了Read
方法：

    var r io.Reader
    r = os.Stdin
    r = bufio.NewReader(r)
    r = new(bytes.Buffer)
    // and so on

要时刻牢记：不管变量 r 指向的具体值是什么，它的类型永远是
io.Reader。再重复一次：Go语言是静态类型语言，变量 r 的静态类型是
io.Reader。

一个非常非常重要的接口类型是空接口，即：

    interface{}

它代表一个空集，没有任何方法。由于任何具体的值都有
零或更多个方法，因此类型为interface{} 的变量能够存储任何值。

有人说，Go的接口是动态类型的。这个说法是错的！接口变量也是静态类型的，它永远只有一个相同的静态类型。如果在运行时它存储的值发生了变化，这个值也必须满足接口类型的方法集合。

由于反射和接口两者的关系很密切，我们必须澄清这一点。

接口变量的表示
--------------

Russ Cox 在2009年写了一篇文章介绍
Go中接口变量的表示形式，具体参考文章末尾的链接“Go语言接口的表示”。这里我们不需要重复所有的细节，只做一个简单的总结。

Interface变量存储一对值：赋给该变量的具体的值、值类型的描述符。更准确一点来说，值就是实现该接口的底层数据，类型是底层数据类型的描述。举个例子：

    var r io.Reader
    tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
    if err != nil {
        return nil, err
    }
    r = tty

在这个例子中，变量 r 在结构上包含一个 (value, type) 对：(tty,
*os.File)。注意：类型* os.File 不仅仅实现了 Read
方法。虽然接口变量只提供 Read
函数的调用权，但是底层的值包含了关于这个值的所有类型信息。所以我们能够做这样的类型转换：

    var w io.Writer
    w = r.(io.Writer)

上面代码的第二行是一个类型断言，它断定变量 r 内部的实际值也继承了
io.Writer接口，所以才能被赋值给 w。赋值之后，w 就指向了 (tty, \*os.File)
对，和变量 r 指向的是同一个 (value, type)
对。不管底层具体值的方法集有多大，由于接口的静态类型限制，接口变量只能调用特定的一些方法。

我们继续往下看：

    var empty interface{}
    empty = w

这里的空接口变量 empty 也包含 (tty, \*os.File)
对。这一点很容易理解：空接口变量可以存储任何具体值以及该值的所有描述信息。

细心的朋友可能会发现，这里没有使用类型断言，因为变量 w 满足
空接口的所有方法（传说中的“无招胜有招”）。在前一个例子中，我们把一个具体值
从 io.Reader 转换为 io.Writer 时，需要显式的类型断言，是因为 io.Writer
的方法集合 不是 io.Reader 的子集。

另外需要注意的一点是，(value, type) 对中的 type 必须是
具体的类型（struct或基本类型），不能是 接口类型。
接口类型不能存储接口变量。

关于接口，我们就介绍到这里，下面我们看看Go语言的反射三定律。

反射第一定律：反射可以将“接口类型变量”转换为“反射类型对象”。
------------------------------------------------------------

注：这里反射类型指 `reflect.Type` 和 `reflect.Value`。

从用法上来讲，反射提供了一种机制，允许程序在运行时检查接口变量内部存储的
(value, type) 对。在最开始，我们先了解下 reflect 包的两种类型：Type 和
Value。这两种类型使访问接口内的数据成为可能。它们对应两个简单的方法，分别是
reflect.TypeOf 和 reflect.ValueOf，分别用来读取接口变量的 reflect.Type
和 reflect.Value 部分。当然，从 reflect.Value 也很容易获取到
reflect.Type。目前我们先将它们分开。

首先，我们下看 reflect.TypeOf：

    package main

    import (
        "fmt"
        "reflect"
    )

    func main() {
        var x float64 = 3.4
        fmt.Println("type:", reflect.TypeOf(x))
    }

这段代码会打印出：

    type: float64

你可能会疑惑：为什么没看到接口？这段代码看起来只是把一个
float64类型的变量 x 传递给
reflect.TypeOf，并没有传递接口。事实上，接口就在那里。查阅一下TypeOf
的文档，你会发现 reflect.TypeOf 的函数签名里包含一个空接口：

    // TypeOf returns the reflection Type of the value in the interface{}.
    func TypeOf(i interface{}) Type

我们调用 reflect.TypeOf(x) 时，x 被存储在一个空接口变量中被传递过去；
然后reflect.TypeOf 对空接口变量进行拆解，恢复其类型信息。

函数 reflect.ValueOf
也会对底层的值进行恢复（这里我们忽略细节，只关注可执行的代码）：

    var x float64 = 3.4
    fmt.Println("value:", reflect.ValueOf(x))

上面这段代码打印出：

    value: <float64 Value>

类型 reflect.Type 和 reflect.Value
都有很多方法，我们可以检查和使用它们。这里我们举几个例子。类型
reflect.Value 有一个方法 Type()，它会返回一个 reflect.Type
类型的对象。Type和 Value都有一个名为 Kind
的方法，它会返回一个常量，表示底层数据的类型，常见值有：Uint、Float64、Slice等。Value类型也有一些类似于Int、Float的方法，用来提取底层的数据。Int方法用来提取
int64, Float方法用来提取 float64，参考下面的代码：

    var x float64 = 3.4
    v := reflect.ValueOf(x)
    fmt.Println("type:", v.Type())
    fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
    fmt.Println("value:", v.Float())

上面这段代码会打印出：

    type: float64
    kind is float64: true
    value: 3.4

还有一些用来修改数据的方法，比如SetInt、SetFloat，在讨论它们之前，我们要先理解“可修改性”（settability），这一特性会在“反射第三定律”中进行详细说明。

反射库提供了很多值得列出来单独讨论的属性。首先是介绍下Value 的 getter 和
setter 方法。为了保证API
的精简，这两个方法操作的是某一组类型范围最大的那个。比如，处理任何含符号整型数，都使用
int64。也就是说 Value 类型的Int 方法返回值为 int64类型，SetInt
方法接收的参数类型也是 int64
类型。实际使用时，可能需要转化为实际的类型：

    var x uint8 = 'x'
    v := reflect.ValueOf(x)
    fmt.Println("type:", v.Type())                            // uint8.
    fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
    x = uint8(v.Uint())                // v.Uint returns a uint64.

第二个属性是反射类型变量（reflection object）的 Kind 方法
会返回底层数据的类型，而不是静态类型。如果一个反射类型对象包含一个用户定义的整型数，看代码：

    type MyInt int
    var x MyInt = 7
    v := reflect.ValueOf(x)

上面的代码中，虽然变量 v 的静态类型是MyInt，不是 int，Kind 方法仍然返回
reflect.Int。换句话说， Kind 方法不会像 Type 方法一样区分 MyInt 和 int。

反射第二定律：反射可以将“反射类型对象”转换为“接口类型变量”。
------------------------------------------------------------

和物理学中的反射类似，Go语言中的反射也能创造自己反面类型的对象。

根据一个 reflect.Value 类型的变量，我们可以使用 Interface
方法恢复其接口类型的值。事实上，这个方法会把 type 和 value
信息打包并填充到一个接口变量中，然后返回。其函数声明如下：

    // Interface returns v's value as an interface{}.
    func (v Value) Interface() interface{}

然后，我们可以通过断言，恢复底层的具体值：

    y := v.Interface().(float64) // y will have type float64.
    fmt.Println(y)

上面这段代码会打印出一个 float64 类型的值，也就是 反射类型变量 v
所代表的值。

事实上，我们可以更好地利用这一特性。标准库中的 fmt.Println 和 fmt.Printf
等函数都接收空接口变量作为参数，fmt
包内部会对接口变量进行拆包（前面的例子中，我们也做过类似的操作）。因此，fmt
包的打印函数在打印 reflect.Value 类型变量的数据时，只需要把 Interface
方法的结果传给 格式化打印程序：

    fmt.Println(v.Interface())

你可能会问：问什么不直接打印 v ，比如 fmt.Println(v)？ 答案是 v 的类型是
reflect.Value，我们需要的是它存储的具体值。由于底层的值是一个
float64，我们可以格式化打印：

    fmt.Printf("value is %7.1e\n", v.Interface())

上面代码的打印结果是：

    3.4e+00

同样，这次也不需要对 v.Interface()
的结果进行类型断言。空接口值内部包含了具体值的类型信息，Printf
函数会恢复类型信息。

简单来说，Interface 方法和 ValueOf
函数作用恰好相反，唯一一点是，返回值的静态类型是 interface{}。

我们重新表述一下：Go的反射机制可以将“接口类型的变量”转换为“反射类型的对象”，然后再将“反射类型对象”转换过去。

反射第三定律：如果要修改“反射类型对象”，其值必须是“可写的”（settable）。
------------------------------------------------------------------------

这条定律很微妙，也很容易让人迷惑。但是如果你从第一条定律开始看，应该比较容易理解。

下面这段代码不能正常工作，但是非常值得研究：

    var x float64 = 3.4
    v := reflect.ValueOf(x)
    v.SetFloat(7.1) // Error: will panic.

如果你运行这段代码，它会抛出抛出一个奇怪的异常：

    panic: reflect.Value.SetFloat using unaddressable value

这里问题不在于值 7.1 不能被寻址，而是因为变量 v
是“不可写的”。“可写性”是反射类型变量的一个属性，但不是所有的反射类型变量都拥有这个属性。

我们可以通过 CanSet 方法检查一个 reflect.Value
类型变量的“可写性”。对于上面的例子，可以这样写：

    var x float64 = 3.4
    v := reflect.ValueOf(x)
    fmt.Println("settability of v:", v.CanSet())

上面这段代码打印结果是：

    settability of v: false

对于一个不具有“可写性”的 Value类型变量，调用 Set
方法会报出错误。首先，我们要弄清楚什么“可写性”。

“可写性”有些类似于寻址能力，但是更严格。它是反射类型变量的一种属性，赋予该变量修改底层存储数据的能力。“可写性”最终是由一个事实决定的：反射对象是否存储了原始值。举个代码例子：

    var x float64 = 3.4
    v := reflect.ValueOf(x)

这里我们传递给 reflect.ValueOf 函数的是变量 x 的一个拷贝，而非 x
本身。想象一下，如果下面这行代码能够成功执行：

    v.SetFloat(7.1)

答案是：如果这行代码能够成功执行，它不会更新 x ，虽然看起来变量 v 是根据
x 创建的。相反，它会更新 x 存在于 反射对象 v 内部的一个拷贝，而变量 x
本身完全不受影响。这会造成迷惑，并且没有任何意义，所以是不合法的。“可写性”就是为了避免这个问题而设计的。

这看起来很诡异，事实上并非如此，而且类似的情况很常见。考虑下面这行代码：

    f(x)

上面的代码中，我们把变量 x 的一个拷贝传递给函数，因此不期望它会改变 x
的值。如果期望函数 f 能够修改变量 x，我们必须传递 x 的地址（即指向 x
的指针）给函数 f，如下：

    f(&x)

你应该很熟悉这行代码，反射的工作机制是一样的。如果你想通过反射修改变量
x，就咬吧想要修改的变量的指针传递给 反射库。

首先，像通常一样初始化变量 x，然后创建一个指向它的 反射对象，名字为 p：

    var x float64 = 3.4
    p := reflect.ValueOf(&x) // Note: take the address of x.
    fmt.Println("type of p:", p.Type())
    fmt.Println("settability of p:", p.CanSet())

这段代码的输出是：

    type of p: *float64
    settability of p: false

反射对象 p 是不可写的，但是我们也不像修改 p，事实上我们要修改的是
\*p。为了得到 p 指向的数据，可以调用 Value 类型的 Elem 方法。Elem
方法能够对指针进行“解引用”，然后将结果存储到反射 Value类型对象 v中：

    v := p.Elem()
    fmt.Println("settability of v:", v.CanSet())

在上面这段代码中，变量 v 是一个可写的反射对象，代码输出也验证了这一点：

    settability of v: true

由于变量 v 代表 x， 因此我们可以使用 v.SetFloat 修改 x 的值：

    v.SetFloat(7.1)
    fmt.Println(v.Interface())
    fmt.Println(x)

上面代码的输出如下：

    7.1
    7.1

反射不太容易理解，reflect.Type 和 reflect.Value
会混淆正在执行的程序，但是它做的事情正是编程语言做的事情。你只需要记住：只要反射对象要修改它们表示的对象，就必须获取它们表示的对象的地址。

结构体（struct）
----------------

在前面的例子中，变量 v
本身并不是指针，它只是从指针衍生而来。把反射应用到结构体时，常用的方式是
使用反射修改一个结构体的某些字段。只要拥有结构体的地址，我们就可以修改它的字段。

下面通过一个简单的例子对结构体类型变量 t 进行分析。

首先，我们创建了反射类型对象，它包含一个结构体的指针，因为后续会修改。

然后，我们设置 typeOfT 为它的类型，并遍历所有的字段。

注意：我们从 struct 类型提取出每个字段的名字，但是每个字段本身也是常规的
reflect.Value 对象。

    type T struct {
        A int
        B string
    }
    t := T{23, "skidoo"}
    s := reflect.ValueOf(&t).Elem()
    typeOfT := s.Type()
    for i := 0; i < s.NumField(); i++ {
        f := s.Field(i)
        fmt.Printf("%d: %s %s = %v\n", i,
            typeOfT.Field(i).Name, f.Type(), f.Interface())
    }

上面这段代码的输出如下：

    0: A int = 23
    1: B string = skidoo

这里还有一点需要指出：变量 T
的字段都是首字母大写的（暴露到外部），因为struct中只有暴露到外部的字段才是“可写的”。

由于变量 s 包含一个“可写的”反射对象，我们可以修改结构体的字段：

    f.Interface())s.Field(0).SetInt(77)
    s.Field(1).SetString("Sunset Strip")
    fmt.Println("t is now", t)

上面代码的输出如下：

    t is now {77 Sunset Strip}

如果变量 s 是通过 t ，而不是 &t 创建的，调用 SetInt 和 SetString
将会失败，因为 t 的字段不是“可写的”。

结论
----

最后再次重复一遍反射三定律：

1.  反射可以将“接口类型变量”转换为“反射类型对象”。

2.  反射可以将“反射类型对象”转换为“接口类型变量”。

3.  如果要修改“反射类型对象”，其值必须是“可写的”（settable）。\
    一旦你理解了这些定律，使用反射将会是一件非常简单的事情。它是一件强大的工具，使用时务必谨慎使用，更不要滥用。

关于反射，我们还有很多内容没有讨论，包括基于管道的发送和接收、内存分配、使用slice和map、调用方法和函数，由于本文已经非常长了，这些话题在后续的文章中介绍。

------------------------------------------------------------------------

原作者 Rob Pike，翻译Oscar

相关链接：

-   原文链接：[https://blog.golang.org/laws-...](https://blog.golang.org/laws-of-reflection)

-   reflect 包：<https://golang.org/pkg/reflect/>

扫码关注微信公众号“深入Go语言”



