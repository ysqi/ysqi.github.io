
---
date: 2016-12-31T11:34:38+08:00
title: "【转载】Go语言设计模式实践:迭代器(Iterator)"
description: ""
disqus_identifier: 1485833678705464854
slug: "【zhuai-zai-】Goyu-yan-she-ji-mo-shi-shi-jian-:die-dai-qi-(Iterator)"
source: "https://segmentfault.com/a/1190000002385729"
tags: 
- 转载 
- 设计模式 
- golang 
topics:
- 编程语言与开发
---

原文：<http://www.cnblogs.com/newgame/p/4061083.html>

关于本系列
----------

决定开个新坑。

这个系列首先是关于Go语言实践的。在项目中实际使用Go语言也有段时间了，一个体会就是不论是官方文档、图书还是网络资料，关于Go语言惯用法（idiom）的介绍都比较少，基本只能靠看标准库源代码自己琢磨，所以我特别想在这方面有一些收集和总结。

然后这个系列也是关于设计模式的。虽然Go语言不是一门面向对象编程语言，但是很多面向对象设计模式所要解决的问题是在程序设计中客观存在的。不管用什么语言，总是要面对和解决这些问题的，只是解决的思路和途径会有所不同。所以我想就以经典的设计模式作为切入点来展开这个系列，毕竟大家对设计模式都很熟悉了，可以避免无中生有想出一些蹩脚的应用场景。

本系列的具体主题会比较灵活，计划主要包括这些方面的话题：

Go语言惯用法。\
设计模式的实现。特别是引入了闭包，协程，DuckType等语言特性后带来的变化。\
设计模式思想的探讨。会有一些吐槽。

不使用迭代器的方案
------------------

首先要指出的是，绝大多数情况下Go程序是不需要用迭代器的。因为内置的slice和map两种容器都可以通过range进行遍历，并且这两种容器在性能方面做了足够的优化。只要没有特殊的需求，通常是直接用这两种容器解决问题。即使不得不写了一个自定义容器，我们几乎总是可以实现一个函数，把所有元素（的引用）拷贝到一个slice之后返回，这样调用者又可以直接用range进行遍历了。

当然某些特殊场合迭代器还是有用武之地。比如迭代器的Next()是个耗时操作，不能一口气拷贝所有元素；再比如某些条件下需要中断遍历。

经典实现
--------

经典实现完全采用面向对象的思路。为了简化问题，下面的例子中容器就是简单的\[\]int，我们在main函数中使用迭代器进行遍历操作并打印取到的值，迭代器的接口设计参考java。

    package main

    import "fmt"

    type Ints []int

    func (i Ints) Iterator() *Iterator {
        return &Iterator{
            data:  i,
            index: 0,
        }
    }

    type Iterator struct {
        data  Ints
        index int
    }

    func (i *Iterator) HasNext() bool {
        return i.index < len(i.data)
    }

    func (i *Iterator) Next() (v int) {
        v = i.data[i.index]
        i.index++
        return v
    }

    func main() {
        ints := Ints{1, 2, 3}
        for it := ints.Iterator(); it.HasNext(); {
            fmt.Println(it.Next())
        }
    }

闭包实现
--------

Go语言支持first class
functions、高阶函数、闭包、多返回值函数。用上这些特性可以换种方式实现迭代器。

初看之下闭包实现与经典实现完全不同，其实从本质上来看，二者区别不大。经典实现中把迭代器需要的数据存在struct中，HasNext()
Next()两个函数定义为Iterator的方法从而和数据绑定了起来；闭包实现中迭代器是一个匿名函数，它所需要的数据i
Ints和index以闭包up
value的形式绑定了起来，匿名函数返回的两个值正好对应经典实现中的Next()和HasNext()。

    package main

    import "fmt"

    type Ints []int

    func (i Ints) Iterator() func() (int, bool) {
        index := 0
        return func() (val int, ok bool) {
            if index >= len(i) {
                return
            }

            val, ok = i[index], true
            index++
            return
        }
    }

    func main() {
        ints := Ints{1, 2, 3}
        it := ints.Iterator()
        for {
            val, ok := it()
            if !ok {
                break
            }
            fmt.Println(val)
        }
    }

channel实现
-----------

这份实现是最go
way的，使用了一个channel在新的goroutine中将容器内的元素依次输出。优点是channel是可以用range接收的，所以调用方代码很简洁；缺点是goroutine上下文切换会有开销，这份实现无疑是最低效的，另外调用方必须接收完所有数据，如果只接收一半就中断掉发送方将永远阻塞。

依稀记得在邮件列表里看到说标准库里有这个用法的例子，刚才去翻了下没找到原帖了:-)

顺便说一下，“在函数中创建一个channel返回，同时创建一个goroutine往channel中塞数据”这是一个重要的惯用法（Channel
Factory pattern，见the way to go
18.8节），可以用来做序列发生器、fan-out、fan-in等。

    package main

    import "fmt"

    type Ints []int

    func (i Ints) Iterator() <-chan int {
        c := make(chan int)
        go func() {
            for _, v := range i {
                c <- v
            }
            close(c)
        }()
        return c
    }

    func main() {
        ints := Ints{1, 2, 3}
        for v := range ints.Iterator() {
            fmt.Println(v)
        }
    }

Do实现
------

这份迭代器实现是最简洁的，代码也很直白，无须多言。如果想加上中断迭代的功能，可以将func(int)改为func(int)bool，Do中根据返回值决定是否退出迭代。

标准库中的container/ring中有Do()用法的例子。

    package main

    import "fmt"

    type Ints []int

    func (i Ints) Do(fn func(int)) {
        for _, v := range i {
            fn(v)
        }
    }

    func main() {
        ints := Ints{1, 2, 3}
        ints.Do(func(v int) {
            fmt.Println(v)
        })
    }

总结
----

Go语言中没有class和继承，不具备完整表达面向对象的能力，不是一门通常意义上的面向对象语言。但是这不妨碍Go语言实现面向对象的思想，利用其语言特性，实现封装、组合、多态都没有问题。\
设计模式的精髓在于思想而不在于类图。编程语言是在不断进步的，类图却一直用几十年前那一张，抛开类图重新审视问题，合理利用语言新特性可以得到更简洁的设计模式实现。

