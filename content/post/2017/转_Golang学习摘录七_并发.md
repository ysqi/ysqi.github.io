
---
date: 2017-02-24T08:31:54+08:00
title: "Golang学习摘录七:并发"
description: ""
disqus_identifier: 1487896314031568564
slug: "Golangxue-xi-zhai-lu-qi-:bing-fa"
source: "http://www.jianshu.com/p/14915a7d0f93"
tags: 
- golang 
categories:
- 编程语言与开发
---

Go使用channel和goroutine开发并行程序。goroutine 是
Go并发能力的核心要素。\
goroutine 是一个普通的函数，只是需要使用关键字 go 作为开头。

    ready("Tea", 2) // 普通函数调用
    go ready("Tea", 2) // ready() 作为 goroutine 运行

Go routine实践

    func ready(w string, sec int) {
      time.Sleep(time.Duration(sec) * time.Second)
      fmt.Println(w,"is ready!")
    }
    func main() {
      go ready("Tea", 2)
      go ready("Coffee", 1)
       fmt.Println("I'm waiting")
       time.Sleep(5 * time.Second)
    }
    // 输出
      I'm waiting //立刻
    Coffee is ready! //1秒后
    Tea is ready! //2秒回

如果不等待goroutine的执行（例如移除第17行），程序会立刻终止，而任何正在执行的goroutine都会停止。为了修复使用channels机制来和goroutine通讯。可以通过channel发送或接受值。这些值只能是特定的类型:channel
类型。\
注意，必须使用 make 创建 channel：

    ci := make(chan int) //创建 channel ci 用于发送和接收整数
    cs := make(chan string) //创建 channel cs 用于字符串
    cf:=make(chan interface{})//channel cf 使用了空接口来满足各种类型

向 channel 发送或接收数据，是通过类似的操作符完 成的:\<−.
具体作用则依赖于操作符的位置:

    ci <− 1 // 发送整数 1 到 channelci
    <−ci // 从 channel ci 接收整数
    i := <−ci // 从 channel ci 接收整数，并保存到 i 中

上面的例子使用channel后变为：

    var c chan int //定义 c 作为 int 型的 channel。就是说:这个 channel 传输整数。注意这个变量是 全局的，这样 goroutine 可以访问它;
    func ready(w string, sec int) {
        time.Sleep(time.Duration(sec) * time.Second)
        fmt.Println(w, "is ready!")
        c <- 1 //发送整数 1 到 channel c;
    }
    func main() {
      c=make(chan int) //初始化c;
      go ready("Tea", 2) //用关键字 go 开始一个 goroutine;
      go ready("Coffee", 1)
      fmt.Println("I'm waiting, but not too long")
      <−c // 等待，直到从 channel 上接收一个值。注意，收到的值被丢弃了;
      <−c// 两个 goroutines，接收两个值。
    }
    // 如果不知道启动了多少个goroutine，可以使用select监听channel上输入的数据
    L: for {
     select { 
     case <−c: 
          i++ 
          if i>1{
            break L 
          }
      }
    }
    // 现在将会一直等待下去。只有当从 channel c 上收到多个响应时才会退出循环 L。

虽然 goroutine 是并发执行的，但是它们并不是并行运行的。如果不告诉 Go
额外的 东西，同一时刻只会有一个 goroutine 执行。利用
runtime.GOMAXPROCS(n) 可以设置 goroutine
并行执行的数量。如果不希望修改任何源代码，同样可以通过设置环境变量
GOMAXPROCS 为目标值。\
当在 Go 中用 ch := make(chan bool) 创建 chennel 时，bool 型的 无缓冲
channel 会 被创建。channel同样可以指定缓冲大小，ch := make(chan bool,
4)，创建了可以存储 4 个元素的 bool型channel。在这个 channel 中，前 4
个元素可以无阻塞的写入。当写入第 5 元素时，代码 将会阻塞，直 到其他
goroutine 从 channel 中读取一些元素，腾出空间。

当 channel 被关闭后，读取端需要知道这个事情。

    x, ok = <−ch

当 ok 被赋值为 true 意味着 channel 尚未被关闭，同时 可以读取数据。否则
ok 被赋值为 false。在这个情况下表示 channel 被关闭。

