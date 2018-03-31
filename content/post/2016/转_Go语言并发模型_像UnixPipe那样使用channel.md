
---
date: 2016-12-31T11:33:13+08:00
title: "Go语言并发模型:像UnixPipe那样使用channel"
description: ""
disqus_identifier: 1485833593095629490
slug: "Goyu-yan-bing-fa-mo-xing-:xiang-Unix-Pipena-yang-shi-yong-channel"
source: "https://segmentfault.com/a/1190000006261218"
tags: 
- channel 
- concurrency 
- golang 
categories:
- 编程语言与开发
---

简介
----

Go语言的并发原语允许开发者以类似于 Unix Pipe 的方式构建数据流水线 (data
pipelines)，数据流水线能够高效地利用 I/O和多核 CPU 的优势。

本文要讲的就是一些使用流水线的一些例子，流水线的错误处理也是本文的重点。

阅读建议
--------

数据流水线充分利用了多核特性，代码层面是基于 channel 类型 和 go 关键字。

channel 和 go
贯穿本文的始终。如果你对这两个概念不太了解，建议先阅读之前公众号发布的两篇文章：Go
语言内存模型(上/下)。

如果你对操作系统中"生产者"和"消费者"模型比较了解的话，也将有助于对本文中流水线的理解。

本文中绝大多数讲解都是基于代码进行的。换句话说，如果你看不太懂某些代码片段，建议补全以后，在机器或play.golang.org
上运行一下。对于某些不明白的细节，可以手动添加一些语句以助于理解。

由于 Go语言并发模型 的英文原文 [Go Concurrency Patterns: Pipelines and
cancellation](https://blog.golang.org/pipelines) 篇幅比较长，本文只包含
理论推导和简单的例子。\
下一篇文章我们会对 "并行MD5" 这个现实生活的例子进行详细地讲解。

什么是 "流水线" (pipeline)?
---------------------------

对于"流水线"这个概念，Go语言中并没有正式的定义，它只是很多种并发方式的一种。这里我给出一个非官方的定义：一条流水线是
是由多个阶段组成的，相邻的两个阶段由 channel 进行连接；\
每个阶段是由一组在同一个函数中启动的 goroutine 组成。在每个阶段，这些
goroutine 会执行下面三个操作：

1.  通过 inbound channels 从上游接收数据

2.  对接收到的数据执行一些操作，通常会生成新的数据

3.  将新生成的数据通过 outbound channels 发送给下游

除了第一个和最后一个阶段，每个阶段都可以有任意个 inbound 和 outbound
channel。\
显然，第一个阶段只有 outbound channel，而最后一个阶段只有 inbound
channel。\
我们通常称第一个阶段为`"生产者"`或`"源头"`，称最后一个阶段为`"消费者"`或`"接收者"`。

首先，我们通过一个简单的例子来演示这个概念和其中的技巧。后面我们会更出一个真实世界的例子。

流水线入门：求平方数
--------------------

假设我们有一个流水线，它由三个阶段组成。

第一阶段是 gen 函数，它能够将一组整数转换为channel，channel
可以将数字发送出去。\
gen 函数首先启动一个 goroutine，该goroutine 发送数字到
channel，当数字发送完时关闭channel。\
代码如下：

    func gen(nums ...int) <-chan int {
        out := make(chan int)
        go func() {
            for _, n := range nums {
                out <- n
            }
            close(out)
        }()
        return out
    }

第二阶段是 sq 函数，它从 channel 接收一个整数，然后返回
一个channel，返回的channel可以发送 接收到整数的平方。\
当它的 inbound channel 关闭，并且把所有数字均发送到下游时，会关闭
outbound channel。代码如下：

    func sq(in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            for n := range in {
                out <- n * n
            }
            close(out)
        }()
        return out
    }

main 函数
用于设置流水线并运行最后一个阶段。最后一个阶段会从第二阶段接收数字，并逐个打印出来，直到来自于上游的
inbound channel关闭。代码如下：

    func main() {
        // 设置流水线
        c := gen(2, 3)
        out := sq(c)

        // 消费输出结果
        fmt.Println(<-out) // 4
        fmt.Println(<-out) // 9
    }

由于 sq 函数的 inbound channel 和 outbound channel
类型一样，所以组合任意个 sq 函数。比如像下面这样使用：

    func main() {
        // 设置流水线并消费输出结果
        for n := range sq(sq(gen(2, 3))) {
            fmt.Println(n) // 16 then 81
        }
    }

如果我们稍微修改一下 gen 函数，便可以模拟
haskell的惰性求值。有兴趣的读者可以自己折腾一下。

流水线进阶：扇入和扇出
----------------------

**扇出**：同一个 channel 可以被多个函数读取数据，直到channel关闭。\
这种机制允许将工作负载分发到一组worker，以便更好地并行使用 CPU 和 I/O。

**扇入**：多个 channel 的数据可以被同一个函数读取和处理，然后合并到一个
channel，直到所有 channel都关闭。

下面这张图对 扇入 有一个直观的描述：

我们修改一下上个例子中的流水线，这里我们运行两个 sq 实例，它们从同一个
channel 读取数据。\
这里我们引入一个新函数 merge 对结果进行"扇入"操作：

    func main() {
        in := gen(2, 3)

        // 启动两个 sq 实例，即两个goroutines处理 channel "in" 的数据
        c1 := sq(in)
        c2 := sq(in)

        // merge 函数将 channel c1 和 c2 合并到一起，这段代码会消费 merge 的结果
        for n := range merge(c1, c2) {
            fmt.Println(n) // 打印 4 9, 或 9 4
        }
    }

merge 函数 将多个 channel 转换为一个 channel，它为每一个 inbound channel
启动一个 goroutine，用于将数据\
拷贝到 outbound channel。\
merge 函数的实现见下面代码 (注意 wg 变量)：

    func merge(cs ...<-chan int) <-chan int {
        var wg sync.WaitGroup
        out := make(chan int)

        // 为每一个输入channel cs 创建一个 goroutine output
        // output 将数据从 c 拷贝到 out，直到 c 关闭，然后 调用 wg.Done
        output := func(c <-chan int) {
            for n := range c {
                out <- n
            }
            wg.Done()
        }
        wg.Add(len(cs))
        for _, c := range cs {
            go output(c)
        }

        // 启动一个 goroutine，用于所有 output goroutine结束时，关闭 out 
        // 该goroutine 必须在 wg.Add 之后启动
        go func() {
            wg.Wait()
            close(out)
        }()
        return out
    }

在上面的代码中，每个 inbound channel 对应一个 `output` 函数。所有
`output` goroutine 被创建以后，merge 启动一个额外的 goroutine，\
这个goroutine会等待所有 inbound channel 上的发送操作结束以后，关闭
outbound channel。

对已经关闭的channel
执行发送操作(ch&lt;-)会导致异常，所以我们必须保证所有的发送操作都在关闭channel之前结束。\
[sync.WaitGroup](http://golang.org/pkg/sync/#WaitGroup)
提供了一种组织同步的方式。\
它保证 merge 中所有 inbound channel (cs ...&lt;-chan int) 均被正常关闭，
output goroutine 正常结束后，关闭 out channel。

停下来思考一下
--------------

在使用流水线函数时，有一个固定的模式：

1.  在一个阶段，当所有发送操作 (ch&lt;-) 结束以后，关闭 outbound channel

2.  在一个阶段，goroutine 会持续从 inbount channel 接收数据，直到所有
    inbound channel 全部关闭

在这种模式下，每一个接收阶段都可以写成 `range` 循环的方式，\
从而保证所有数据都被成功发送到下游后，goroutine能够立即退出。

在现实中，阶段并不总是接收所有的 inbound
数据。有时候是设计如此：接收者可能只需要数据的一个子集就可以继续执行。\
更常见的情况是：由于前一个阶段返回一个错误，导致该阶段提前退出。\
这两种情况下，接收者都不应该继续等待后面的值被传送过来。

我们期望的结果是：`当后一个阶段不需要数据时，上游阶段能够停止生产。`

在我们的例子中，如果一个阶段不能消费所有的 inbound
数据，试图发送这些数据的 goroutine 会永久阻塞。看下面这段代码片段：

        // 只消费 out 的第一个数据
        out := merge(c1, c2)
        fmt.Println(<-out) // 4 or 9
        return
        // 由于我们不再接收 out 的第二个数据
        // 其中一个 goroutine output 将会在发送时被阻塞
    }

显然这里存在资源泄漏。一方面goroutine
消耗内存和运行时资源，另一方面goroutine 栈中的堆引用会阻止 gc
执行回收操作。\
既然goroutine 不能被回收，那么他们必须自己退出。

我们重新整理一下流水线中的不同阶段，保证在下游阶段接收数据失败时，上游阶段也能够正常退出。\
一个方式是使用带有缓冲的管道作为 outbound
channel。缓存可以存储固定个数的数据。\
如果缓存没有用完，那么发送操作会立即返回。看下面这段代码示例：

    c := make(chan int, 2) // 缓冲大小为 2
    c <- 1  // 立即返回
    c <- 2  // 立即返回
    c <- 3  // 该操作会被阻塞，直到有一个 goroutine 执行 <-c，并接收到数字 1

如果在创建 channel 时就知道要发送的值的个数，使用buffer就能够简化代码。\
仍然使用求平方数的例子，我们对 gen
函数进行重写。我们将这组整型数拷贝到一个\
缓冲 channel中，从而避免创建一个新的 goroutine：

    func gen(nums ...int) <-chan int {
        out := make(chan int, len(nums))
        for _, n := range nums {
            out <- n
        }
        close(out)
        return out
    }

回到 流水线中被阻塞的 goroutine，我们考虑让 merge 函数返回一个缓冲管道：

    func merge(cs ...<-chan int) <-chan int {
        var wg sync.WaitGroup
        out := make(chan int, 1) // 在本例中存储未读的数据足够了
        // ... 其他部分代码不变 ...

尽管这种方法解决了这个程序中阻塞
goroutine的问题，但是从长远来看，它并不是好办法。\
缓存大小选择为1 是建立在两个前提之上：

1.  我们已经知道 merge 函数有两个 inbound channel

2.  我们已经知道下游阶段会消耗多少个值

这段代码很脆弱。如果我们在传入一个值给 gen
函数，或者下游阶段读取的值变少，goroutine\
会再次被阻塞。

为了从根本上解决这个问题，我们需要提供一种机制，让下游阶段能够告知上游发送者停止接收的消息。\
下面我们看下这种机制。

显式取消 (Explicit cancellation)
--------------------------------

当 main 函数决定退出，并停止接收 out
发送的任何数据时，它必须告诉上游阶段的 goroutine 让它们放弃\
正在发送的数据。 main 函数通过发送数据到一个名为 done
的channel实现这样的机制。 由于有两个潜在的\
发送者被阻塞，它发送两个值。如下代码所示：

    func main() {
        in := gen(2, 3)

        // 启动两个运行 sq 的goroutine
        // 两个goroutine的数据均来自于 in
        c1 := sq(in)
        c2 := sq(in)

        // 消耗 output 生产的第一个值
        done := make(chan struct{}, 2)
        out := merge(done, c1, c2)
        fmt.Println(<-out) // 4 or 9

        // 告诉其他发送者，我们将要离开
        // 不再接收它们的数据
        done <- struct{}{}
        done <- struct{}{}
    }

发送数据的 goroutine 使用一个 select 表达式代替原来的操作，select
表达式只有在接收到 out 或 done\
发送的数据后，才会继续进行下去。 done 的值类型为 struct{}
，因为它发送什么值不重要，重要的是它发送没发送：\
接收事件发生意味着 channel out 的发送操作被丢弃。 goroutine output 基于
inbound channel c 继续执行\
循环，所以上游阶段不会被阻塞。(后面我们会讨论如何让循环提前退出)。 使用
done channel 方式实现的merge 函数如下：

    func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
        var wg sync.WaitGroup
        out := make(chan int)

        // 为 cs 的的每一个 输入channel
        // 创建一个goroutine。output函数将
        // 数据从 c 拷贝到 out，直到c关闭，
        // 或者接收到 done 信号；
        // 然后调用 wg.Done()
        output := func(c <-chan int) {
            for n := range c {
                select {
                case out <- n:
                case <-done:
                }
            }
            wg.Done()
        }
        // ... the rest is unchanged ...

这种方法有一个问题：每一个下游的接收者需要知道潜在被阻上游发送者的个数，然后向这些发送者发送信号让它们提前退出。\
时刻追踪这些数目是一项繁琐且易出错的工作。

我们需要一种方式能够让未知数目、且个数不受限制的goroutine
停止向下游发送数据。在Go语言中，我们可以通过关闭一个\
channel
实现，因为`在一个已关闭 channel 上执行接收操作(<-ch)总是能够立即返回，返回值是对应类型的零值`。关于这点的细节，点击[这里](https://golang.org/ref/spec#Receive_operator)查看。

换句话说，我们只要关闭 done
channel，就能够让解开对所有发送者的阻塞。对一个管道的关闭操作事实上是对所有接收者的广播信号。

我们把 done channel 作为一个参数传递给每一个 流水线上的函数，通过 defer
表达式声明对 done channel的关闭操作。\
因此，所有从 main 函数作为源头被调用的函数均能够收到 done
的信号，每个阶段都能够正常退出。 使用 done
对main函数重构以后，代码如下：

    func main() {
        // 设置一个 全局共享的 done channel，
        // 当流水线退出时，关闭 done channel
        // 所有 goroutine接收到 done 的信号后，
        // 都会正常退出。
        done := make(chan struct{})
        defer close(done)

        in := gen(done, 2, 3)

        // 将 sq 的工作分发给两个goroutine
        // 这两个 goroutine 均从 in 读取数据
        c1 := sq(done, in)
        c2 := sq(done, in)

        // 消费 outtput 生产的第一个值
        out := merge(done, c1, c2)
        fmt.Println(<-out) // 4 or 9

        // defer 调用时，done channel 会被关闭。
    }

现在，流水线中的每个阶段都能够在 done channel 被关闭时返回。merge
函数中的 output 代码也能够顺利返回，因为它知道 done
channel关闭时，上游发送者 sq 会停止发送数据。 在 defer
表达式执行结束时，所有调用链上的 output 都能保证 wg.Done() 被调用：

    func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
        var wg sync.WaitGroup
        out := make(chan int)

        // 为 cs 的每一个 channel 创建一个 goroutine
        // 这个 goroutine 运行 output，它将数据从 c
        // 拷贝到 out，直到 c 关闭，或者 接收到 done
        // 的关闭信号。人啊后调用 wg.Done()
        output := func(c <-chan int) {
            defer wg.Done()
            for n := range c {
                select {
                case out <- n:
                case <-done:
                    return
                }
            }
        }
        // ... the rest is unchanged ...

同样的原理， done channel 被关闭时，sq
也能够立即返回。在defer表达式执行结束时，所有调用链上的 sq 都能保证 out
channel 被关闭。代码如下：

    func sq(done <-chan struct{}, in <-chan int) <-chan int {
        out := make(chan int)
        go func() {
            defer close(out)
            for n := range in {
                select {
                case out <- n * n:
                case <-done:
                    return
                }
            }
        }()
        return out
    } 

这里，我们给出几条构建流水线的指导：

1.  当所有发送操作结束时，每个阶段都关闭自己的 outbound channels

2.  每个阶段都会一直从 inbound channels 接收数据，直到这些 channels
    被关闭，或发送者解除阻塞状态。

流水线通过两种方式解除发送者的阻塞：

1.  提供足够大的缓冲保存发送者发送的数据

2.  接收者放弃 channel 时，显式地通知发送者。

结论
----

本文介绍了Go
语言中构建数据流水线的一些技巧。流水线的错误处理比较复杂，流水线的每个阶段都可能阻塞向下游发送数据，\
下游阶段也可能不再关注上游发送的数据。上面我们介绍了通过关闭一个channel，向流水线中的所有
goroutine 发送一个 "done" 信号；也定义了\
构建流水线的正确方法。

下一篇文章，我们将通过一个 并行 md5
的例子来说明本文所讲的一些理念和技巧。

原作者 Sameer Ajmani，翻译 Oscar

下期预告：Go语言并发模型：以并行md5计算为例。[英文原文链接](https://blog.golang.org/pipelines)

相关链接
--------

1.  原文链接：[https://blog.golang.org/pipel...](https://blog.golang.org/pipelines)

2.  Go并发模型：[http://talks.golang.org/2012/...](http://talks.golang.org/2012/concurrency.slide#1)

3.  Go高级并发模型：[http://blog.golang.org/advanc...](http://blog.golang.org/advanced-go-concurrency-patterns)

扫码关注微信公众号“深入Go语言”



