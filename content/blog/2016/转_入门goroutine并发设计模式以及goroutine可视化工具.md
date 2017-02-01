
---
date: 2016-12-31T11:33:00+08:00
title: "入门goroutine并发设计模式以及goroutine可视化工具"
description: ""
disqus_identifier: 1485833580992901728
slug: "ru-men-goroutinebing-fa-she-ji-mo-shi-yi-ji-goroutineke-shi-hua-gong-ju"
source: "https://segmentfault.com/a/1190000007111208"
tags: 
- concurrency 
- golang 
topics:
- 编程语言与开发
---

Daisy-Chain
-----------

首先，为了防止过于枯燥，我先列出我最喜欢的一个模式：Daisy-Chain。这个模式比较复杂，对go的并发编程不太熟悉的同学，可以先看下面的模式。然后回过头来看这个。

daisy
chain会创建很多channel，然后把这些channel首尾相接级联起来，组成一条单向链，每个channel都在处理不同的子任务，最后的结果在链的末端输出。这是在2012年的[golang
talks中由Rob
Pike](https://talks.golang.org/2012/concurrency.slide#39)提出的：

    func f(left, right chan int) {
        // 这个函数就把right的输出和left的输入联系起来了。
        left <- 1 + <-right
    }

    func main() {
        const n = 10000
        leftmost := make(chan int)
        right := leftmost
        left := leftmost
        // 创建长度为n的daisy链
        for i := 0; i < n; i++ {
            right = make(chan int)
            go f(left, right)
            left = right
        }
        // 在链的最右端输入1，那么最左端就会得到10001
        go func(c chan int) { c <- 1 }(right)
        fmt.Println(<-leftmost)
    }

整个过程类似下图：\

那么这个模式有什么用呢？它可以用来处理迭代算法，使得部分迭代运算并发执行。只要迭代的每个阶段都是相互独立的即可。比如，计算质数：

    import (
           "fmt"
           "os"
           "runtime/trace"
           "time"
    )
    func Generate(ch chan<- int) {
           for i := 2; ; i++ {
               ch <- i
               // 这是为了方便gotrace绘图
               time.Sleep(10 * time.Millisecond)
           }
    }

    func Filter(ch <-chan int, out chan<- int, prime int) {
           for {
               i := <-ch
               if i%prime != 0 {
                   out <- i
               } else {
                   fmt.Printf("[%d] filter out %d\n", prime, i)
               }
           }
    }

    func main() {
        // 这些也是gotrace要求插入的代码。下同
        trace.Start(os.Stderr)
           ch := make(chan int)
           go Generate(ch)
           for i := 0; i < 10; i++ {
               prime := <-ch   // step1
               fmt.Println(prime)
               out := make(chan int)   // step2 
               go Filter(ch, out, prime)  // step3
               ch = out   // step4
           }
           trace.Stop()
    }

仔细分析上面的代码，它的功能就是输出前10个正整数质数。至于细节就让我们一步步分析看看：\
首先，Generate从2开始遍历正整数，并且在一开始就被放入goroutine里了。结果会放在`ch`里；\
然后，在main中启动一个for循环，在循环的每个step1，都会从`ch`中读出一个质数。2当然是质数，但是后面每一步从ch中读取的都是质数吗？且看下面的代码。\
然后，step2会创建一个新channel
`out`（类似上例的right），ch和它作为输入和输出创建一个Filter的goroutine，专门过滤能被step1的prime整除的数。所以在`out`中输出的都是不会被prime整除的数。\
最后在关键的step4，`out`变成下一个`ch`。相当于增加了一节chain的长度。而ch在每个循环中输出的第一个数，都是被**之前**的所有**质数**无法整除的数，即下一个质数。\
输出日志如下：

    2
    3
    [2] filter out 4
    5
    [2] filter out 6
    7
    [2] filter out 8
    [3] filter out 9
    [2] filter out 10
    11
    [2] filter out 12
    13
    [2] filter out 14
    [3] filter out 15
    [2] filter out 16
    17
    [2] filter out 18
    19
    [2] filter out 20
    [3] filter out 21
    [2] filter out 22
    23
    [2] filter out 24
    [5] filter out 25
    [2] filter out 26
    [3] filter out 27
    [2] filter out 28
    29

为了更直观的展示整个过程，我用divan大神的[gotrace](https://github.com/divan/gotrace)工具画出了goroutine的3d交互图：

其中每个红色竖线表示一个goroutine，时间轴是从上到下的，所以红线越长表示goroutine持续时间越长，也说明它生成的越早。\
可以看到，最早的一个goroutine获得的数字是3，4，……
29，因为2已经被输出了，所以是3到29，然后下一个goroutine获得的就是5，7，9，……
29，因为3被输出，而偶数都被过滤了。以此类推，最后输出的就是前10个质数。\
需要指出的是，这个算法并不是最高效的，但却是非常优雅的。

关于gotrace的安装和使用，请移步[这里](https://github.com/divan/gotrace)。我是根据他的方法给go1.6.3打了补丁后，就能使用了。

好了，下面我们**换些基础的模式**讲一下：

Ping-pong
---------

顾名思义，就是由2个goroutine相互踢皮球组成的模式。是在[2013年的golang
talks](https://talks.golang.org/2013/advconc.slide#6)中提出的。尽管它非常简单，但是却方便我们理解go的并发编程概念。

代码如下：\
用一个int表示ball（球），管道表示table（桌子），两个goroutine就是2个运动员,
分别编号为1和2。

    func main() {
        var Ball int
        table := make(chan int)
        go player("2", table)
        go player("1", table)

        // 首先把球放到“桌上”
        table <- Ball
        time.Sleep(1 * time.Second)
        // 1s后比赛结束……
        <-table
    }

    func player(id string, table chan int) {
        for {
            ball := <-table
            log.Printf("%s got ball[%d]\n", id, ball)
            time.Sleep(50 * time.Millisecond)
            log.Printf("%s bounceback ball[%d]\n", id, ball)
            ball++
            table <- ball
        }
    }

输出如下：

    1 got ball[0]
    1 bounceback ball[0]
    2 got ball[1]
    2 bounceback ball[1]
    1 got ball[2]
    1 bounceback ball[2]
    2 got ball[3]
    2 bounceback ball[3]
    1 got ball[4]
    1 bounceback ball[4]
    2 got ball[5]
    2 bounceback ball[5]

代码简洁易懂，很好理解（看不懂的同学请不要拍我）。\
下面，我们增加一位选手，让3个运动员一块打球

        go player("2", table)
        go player("3", table)
        go player("1", table)

这下子热闹了，输出如下：

    1 got ball[0]
    1 bounceback ball[0]
    2 got ball[1]
    2 bounceback ball[1]
    3 got ball[2]
    3 bounceback ball[2]
    1 got ball[3]
    1 bounceback ball[3]
    2 got ball[4]
    2 bounceback ball[4]
    3 got ball[5]
    3 bounceback ball[5]
    1 got ball[6]
    1 bounceback ball[6]
    2 got ball[7]
    2 bounceback ball[7]
    3 got ball[8]
    3 bounceback ball[8]

看3个人有条不紊的相互击球。此时处女座一定非常满意，但是对于习惯了并发随机性的程序员来说，这实在有些过于美好：为什么它们的顺序如此协调，为什么1总是给2，2给3，3给1，而不是其他顺序呢？

划重点了啊：

> The answer is because Go runtime holds waiting [FIFO
> queue](https://github.com/golang/go/blob/master/src/runtime/chan.go#L34)
> for receivers, that is goroutines ready to receive on the particular
> channel

即，对于接收channel内容的goroutines来说，go的runtime会把它们分配到一个[FIFO队列](https://github.com/golang/go/blob/master/src/runtime/chan.go#L34)中，所以这些goroutines只能按照既定的顺序接收channel的内容，而不会弄乱。所以即使创建上百个palyers，顺序依然是固定的。go实在是太贴心了，有不有！

Fan-In
------

也叫“扇入”，应该是并发编程里面比较普通的一个模式了。fan-in会从多个管道读取输入，并汇总到一个channel输出，形象的比喻如下图：\
\
示例代码如下

    import (
           "fmt"
           "math/rand"
           "os"
           "runtime/trace"
           "time"
    )

    func main() {
           trace.Start(os.Stderr)
           c := fanIn(boring(1), boring(2))
           for i := 0; i < 10; i++ {
               fmt.Println(<-c)
           }
           fmt.Println("You're both boring; I'm leaving.")
           trace.Stop()
    }

    func fanIn(input1, input2 <-chan int) <-chan int {
           c := make(chan int)
           go func() { for {c <- <-input1} }()
           go func() { for {c <- <-input2} }()
           return c
    }

    func boring(msg int) <-chan int {
           c := make(chan int)
           go func() { // We launch the goroutine from inside the function.
               for i := 0; ; i++ {
                   c <- msg*1000 + i
                   time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
               }
           }()
           return c // Return the channel to the caller.
    }

输出为：\
2000\
2001\
1001\
2002\
1002\
2003\
1003\
2004\
1004\
gotrace输出为（注意这是两次独立的运行结果）：\
\
可以看到，两次的结果都汇入了main线程，并且顺序输出，没有丢失数据，也没有死锁。

当然，简单的情况，用select也可以。

select设计的目的就是在channel中间通讯，谁的数据先到达，哪个case分支先执行。

    c1 := boring(1)
    c2 := boring(2)
    for i := 0; i < 10; i++ {
        select {
        case v := <-c1:
            fmt.Println(v)
        case v := <-c2:
            fmt.Println(v)
        }
    }

Workers
-------

也叫FanOut(扇出)，和扇入模式相反，工作模式是一个管道分发任务，多个goroutines来执行。\
示例代码如下：

    import (
        "fmt"
        "os"
        "runtime/trace"
        "sync"
        "time"
    )

    func worker(ch <-chan int, wg *sync.WaitGroup) {
        defer wg.Done()
        for {
            task, ok := <-ch
            if !ok {
                return
            }
            time.Sleep(20 * time.Millisecond)
            fmt.Println("processing task", task)
        }
    }

    func pool(wg *sync.WaitGroup, workers, tasks int) {
        ch := make(chan int)

        for i := 0; i < workers; i++ {
            time.Sleep(1 * time.Millisecond)
            // spawn出很多worker线程
            go worker(ch, wg)
        }

        for i := 0; i < tasks; i++ {
            time.Sleep(10 * time.Millisecond)
            // 开始分发任务，被激活的workers开始工作了
            ch <- i
        }

        close(ch)
    }
    func main() {
        trace.Start(os.Stderr)
        var wg sync.WaitGroup
        wg.Add(36)
        go pool(&wg, 36, 36)
        wg.Wait()
        trace.Stop()
    }

代码略长，但是逻辑其实非常清晰。我在注释中也稍作了说明。\
注意（**划重点了**），`close(ch)`在这里很关键，它确定了每个worker退出的节点。当channel中的内容为空，同时它已经被close时，`task, ok := <- ch`返回的ok==false，此时通知worker退出，wg标记完成，当所有的worker都完成时，wg.Wait()完成，转入下一行执行。\
**在golang中，main不会自动等待所有子进程完成**，如果没有退出检查，main进程会闪退，所有的子进程也会随之强制退出，所以在main里必须有退出检测机制，前几个例子我们使用的是time.Sleep和for循环，这里我们使用了WaitGroup。

gotrace结果如下：\
\
圆柱体中心就是main进程中生成的pool进程，围绕它的是36个worker进程。蓝色箭头表示pool每隔10ms分发的任务，它们都被worker处理了。

Servers
-------

server模式和fan\_out类似，只不过它的worker线程是按需生成的，并且工作处理完毕后就释放。所以这种模式常应用到网站服务器上。在主进程中，有一个for循环，Accept函数一直阻塞着循环的进行，一旦有新的请求过来，Accept就会生成一个connection，然后主进程就创建一个子进程处理这个connection以及其他逻辑。

示例代码如下：

    import (
           "fmt"
           "net"
           "os"
           "runtime/trace"
           "time"
    )

    func handler(c net.Conn, ch chan int) {
       ch <- len(c.RemoteAddr().String())

       time.Sleep(10 * time.Microsecond)
       c.Write([]byte("ok"))
       c.Close()
    }

    func logger(ch chan int) {
           for {
               time.Sleep(1500 * time.Millisecond)
               fmt.Println(<-ch)
           }
    }

    func server(l net.Listener, ch chan int) {
           for {
               c, err := l.Accept()
               if err != nil {
                   continue
               }
               go handler(c, ch)
           }
    }

    func main() {
           trace.Start(os.Stderr)

           l, err := net.Listen("tcp", ":5000")
           if err != nil {
               panic(err)
           }
           ch := make(chan int)
           go logger(ch)
           go server(l, ch)
           time.Sleep(10 * time.Second)
           trace.Stop()
    }

可以看到，主进程生成了一个tcp连接，启动了server和logger两个子进程。server用来监听外网的请求，一旦请求过来，就会生成一个handler进程，用来处理connection。同时，handler还会通过管道和logger通讯，logger负责异步记录相应日志。

这个程序运行时的输入需要模拟外部请求来产生，为此我写了一个脚本：

    #!/bin/sh
    i=0
    while [[ $i -lt 20 ]];
    do
        # 通过nc发起tcp请求。每秒请求一次
        echo "hello "$i | nc localhost 5000
        sleep 1
        ((++i))
    done

运行时，先启动这个脚本，然后启动server或gotrace。

gotrace的运行结果如下：\

可以看到，尽管程序运行了10s，但是只处理了6个请求。这是因为logger占用了管道太长时间，使得handler的运行时间也延长到了1.5s以上。

为了解决这个问题，我们正好借助上面介绍的Worker模式，提高logger的并发性。

### Server + Worker

    import (
               "fmt"
               "net"
               "os"
               "runtime/trace"
               "time"
    )
    func handler(c net.Conn, ch chan int) {
           ch <- 0
           time.Sleep(50 * time.Microsecond)
           c.Write([]byte("ok"))
           c.Close()
    }

    func logger(wch chan int) {
           for {
               fmt.Println(<-wch)
               // 这里主要耗时
               time.Sleep(1500 * time.Millisecond)
           }
    }

    func pool(ch chan int, n int) {
           wch := make(chan int)
           for i := 0; i < n; i++ {
               go logger(wch)
           }
           for {
               wch <- <-ch
           }
    }

    func server(l net.Listener, ch chan int) {
           for {
               c, err := l.Accept()
               if err != nil {
                   continue
               }
               go handler(c, ch)
           }
    }

    func main() {
           trace.Start(os.Stderr)

           l, err := net.Listen("tcp", ":5000")
           if err != nil {
               panic(err)
           }
           ch := make(chan int)
           go pool(ch, 36)
           go server(l, ch)
           time.Sleep(10 * time.Second)
           trace.Stop()
    }

其中pool函数跟上例类似，就是生成（spawn）很多worker，然后handle中生成的数据会先进入pool，由pool再分配给这些workers。

3D图如下：\
\
可以看到，此时server正好处理了10个请求。不再被logger拖延了。

Concurrency & Parallelism
-------------------------

注意我的题目是并发（concurrent）设计模式。那么并发和并行到底啥区别？？

-   Concurrency: A condition that exists when at least two threads are
    making progress. A more generalized form of parallelism that can
    include time-slicing as a form of virtual parallelism.\
    Concurrency（并发性）：是一种广义的并行。在concurrence的语境下，两个线程/任务可以表面上看起来“像是”并行，但其实机器只有一个核，它们只是分享了时间块。当然，在多核的情况下，它可以是并行的。

<!-- -->

-   Parallelism（并行性）：A condition that arises when at least two
    threads are executing simultaneously.\
    这个就是狭义的并行，即线程、任务必须是同时进行的，否则不算parallelism。



