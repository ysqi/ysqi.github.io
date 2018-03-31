
---
date: 2016-12-31T11:33:07+08:00
title: "golangdefer使用小结与注意要点"
description: ""
disqus_identifier: 1485833587354038172
slug: "golang-defer-shi-yong-xiao-jie-yu-zhu-yi-yao-dian"
source: "https://segmentfault.com/a/1190000006823652"
tags: 
- defer 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

关于延时调用函数(Deferred Function Calls)
-----------------------------------------

延时调用函数的语法如下:

    defer func_name(param-list)

当一个函数调用前有关键字 **defer** 时,
那么这个函数的执行会推迟到包含这个 defer 语句的函数即将返回前才执行.
例如:

    func main() {
        defer fmt.Println("Fourth")
        fmt.Println("First")
        fmt.Println("Third")
    }

最后打印顺序如下:

    First
    Second
    Third

`需要注意的是, defer 调用的函数参数的值 defer 被定义时就确定了.`\
例如:

    i := 1
    defer fmt.Println("Deferred print:", i)
    i++
    fmt.Println("Normal print:", i)

打印的内容如下:

    Normal print: 2
    Deferred print: 1

因此我们知道, 在 "defer fmt.Println("Deferred print:", i)" 调用时, i
的值已经确定了, 因此相当于 **defer fmt.Println("Deferred print:", 1)**
了.\
`需要强调的时, defer 调用的函数参数的值在 defer 定义时就确定了, 而 defer 函数内部所使用的变量的值需要在这个函数运行时才确定.`
例如:

    func f1() (r int) {
        r = 1
        defer func() {
            r++
            fmt.Println(r)
        }()
        r = 2
        return
    }

    func main() {
        f1()
    }

上面的例子中, 最终打印的内容是 "3", 这是因为在 "r = 2" 赋值之后, 执行了
defer 函数, 因此在这个函数内, r 的值是2了, 自增后变为3.

### defer 顺序

如果有多个defer 调用, 则调用的顺序是先进后出的顺序, 类似于入栈出栈一样:

    func main() {
        defer fmt.Println(1)
        defer fmt.Println(2)
        defer fmt.Println(3)
        defer fmt.Println(4)
    }

最先执行的是 "fmt.Println(4)", 接着是 "fmt.Println(3)" 依次类推,
最后的输出如下:

    4
    3
    2
    1

### defer 注意要点

`defer 函数调用的执行时机是外层函数设置返回值之后,  并且在即将返回之前`.\
例如:

    func f1() (r int) {
        defer func() {
            r++
        }()
        return 0
    }
    func main() {
        fmt.Println(f1())
    }

上面 **fmt.Println(f1())** 打印的是什么呢? 很多朋友可能会认为打印的是0,
但是正确答案是 1. 这是为什么呢?\
要弄明白这个问题, 我们需要牢记两点

-   `defer 函数调用的执行时机是外层函数设置返回值之后,  并且在即将返回之前`

-   `return XXX 操作并不是原子的.`

我们将上面的例子改写一下大家就很明白了:

    func f1() (r int) {
        defer func() {
            r++
        }()
        r = 0
        return
    }

当进行赋值操作 "r = 0" 后, 才调用 defer 函数, 最后才是返回语句.\
因此上面的代码等效于:

    func f1() (r int) {
        r = 0
        func() {
            r++
        }()
        return
    }

接下来我们再来看一个更有意思的例子:

    func double(x int) int {
        return x + x
    }

    func triple(x int) (r int) {
        defer func() {
            r += x
        }()
        return double(x)
    }

    func main() {
        fmt.Println(triple(3))
    }

如果我们已经理解了上面所说的内容的话, 那么 triple 函数就很好理解了,
它实际上是:

    func triple(x int) (r int) {
        r = double(x)
        func() {
            r += x
        }()
        return
    }

### defer 表达式的使用场景

defer 通常用于 open/close, connect/disconnect, lock/unlock
等这些成对的操作, 来保证在任何情况下资源都被正确释放. 在这个角度来说,
defer 操作和 Java 中的 try ... finally 语句块有异曲同工之处.\
例如:

    var mutex sync.Mutex
    var count = 0

    func increment() {
        mutex.Lock()
        defer mutex.Unlock()
        count++
    }

在increment 函数中, 我们为了避免竞态条件的出现, 而使用了 Mutex 进行加锁.
而在进行并发编程时, 加锁了却忘记(或某种情况下 unlock 没有被执行),
往往会造成灾难性的后果. 为了在任意情况下, 都要保证在加锁操作后,
都进行对应的解锁操作, 我们可以使用 defer 调用解锁操作.

> 本文由 yongshun 发表于个人博客, 采用署名-非商业性使用-相同方式共享 3.0
> 中国大陆许可协议.\
> 非商业转载请注明作者及出处. 商业转载请联系作者本人\
> Email: yongshun1228@gmail.com\
> 本文标题为: golang defer 使用小结与注意要点\
> 本文链接为: segmentfault.com/a/1190000006823652

