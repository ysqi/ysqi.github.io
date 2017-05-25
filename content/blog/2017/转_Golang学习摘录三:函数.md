
---
date: 2017-02-24T08:31:55+08:00
title: "Golang学习摘录三:函数"
description: ""
disqus_identifier: 1487896315644908074
slug: "Golangxue-xi-zhai-lu-san-:han-shu"
source: "http://www.jianshu.com/p/98e2977ef4d1"
tags: 
- golang 
topics:
- 编程语言与开发
---

函数定义
--------

    type mytype int    // 新的类型
    func (p mytype) funcname(q int) (r,s int) {return 0,0}

作用域
------

在 Go
中,定义在函数外的变量是全局的,那些定义在函数内部的变量,对于函数来说
是局部的。如果命名覆盖——一个局部变量与一个全局变量有相同的名字——在函数
执行的时候,局部变量将覆盖全局变量。

多值返回
--------

`func (file *File) Write(b []byte) (n int, err error)`\
Go得函数可以返回多个值

命名返回值
----------

Go
函数的返回值或者结果参数可以指定一个名字,并且像原始的变量那样使用,就像
输入参数那样。如果对其命名,在函数开始时,它们会用其类型的零值初始化。如果
函数在不加参数的情况下执行了 return 语句,结果参数会返回。\
例：

    func ReadFull(r Reader, buf []byte) (n int, err error) {
      for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf) n += nr
        buf = buf[nr:len(buf)]
      }
      return
     }

延迟代码defer
-------------

在 defer 后指定的 函数会在函数退出前调用。

    func ReadWrite() bool {
      file.Open("file")
      defer file.Close()      // file.Close()被添加到了defer列表
      if failureX {
        return false
      }
      if failureY {
        return false
       } 
       return true
    }

可以将多个函数放入 “延迟列表”中,例如：

    for i:=0; i<5; i++ {
    defer fmt.Printf("%d ", i)
    }
    // 延迟的函数是按照后进先出(LIFO)的顺序执行,所以上面的代码打印:4 3 2 1 0。

利用 defer 甚至可以修改返回值,假设正在使用命名结果参数和函数符号。

    defer func() {
    /* ... */
    }()  // ()在这里是必须的

或者这个例子,更加容易了解为什么,以及在哪里需要括号:

    defer func(x int) { /* ... */
    }(5) ／／ 为输入参数 x 赋值 5

在这个(匿名)函数中,可以访问任何命名返回参数:

    func f() (ret int) { //ret初始化为零 
      defer func() {
        ret++  //ret增加为1
      }() 
      return 0    // 返回的是1而不是0
    }

变参
----

接受不定数量的参数的函数叫做变参函数。定义函数使其接受变参:\
`func myfunc(arg ...int) { }`\
arg ...int 告诉 Go 这个函数接受不定数量的参数。注意,这些参数的类型全部是
int。\
在函数体中,变量 arg 是一个 int 类型的 slice:

    for _, n := range arg {
      fmt.Printf("And the number is: %d\n", n)
    }

如果不指定变参的类型,默认是空的接口 interface{}(参阅第 5
章)。假设有另一\
个变参函数叫做 myfunc2,下面的例子演示了如何向其传递变参:

    func myfunc(arg ...int) { myfunc2(arg...) ← 按原样传递
      myfunc2(arg[:2]...) ← 传递部分
    }

函数作为值
----------

Go 中函数也是值，函数可以赋值给变量：

    func main() {
      a := func() {      // 定义一个匿名函数，并赋值给a
        println("Hello")
      }
      a()  // 调用函数
    }

回调
----

由于函数也是值,所以可以很容易的传递到其他函数里,然后可以作为回调。

    func printit(x int) { // 函数无返回值 
      fmt.Printf("%v\n", x) // 仅仅打印
    }
    func callback(y int, f func(int)) { // f 将会保存函数 
      f(y) // 调用回调函数 f 输入变量 y
    }

恐慌(Panic)和恢复(Recover)
--------------------------

Panic：是一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函数F调用panic，函数F的执行被中断，并且F中的延迟函数会正常执行，然后F返回到调用它的地方。在调用的地方，F的行为就像调用了panic。这一过程继续向上，直到程序崩溃时所有goroutine返回。\
恐慌可以直接调用panic产生。也可以由运行时错误产生，例如访问越界的数组。\
Recover：是一个内建函数，可以让进入令人恐慌的流程中的goroutine恢复过来。recover仅在延迟函数中有效。\
在正常的执行过程中，调用recover会返回nil并且没有其他任何效果。如果当期的goroutine陷入恐慌，调用recover可以捕获到panic的输入值，并且恢复正常的执行。\
例：

    这个函数检查作为其参数的函数在执行时是否会产生 panic c: .
     func throwsPanic(f func()) (b bool) {//定义一个新函数 throwsPanic 接受一个函数作为参数(参看 “函数作为值”)。函 数 f 产生 panic,就返回 true,否则返回 false;
      defer func() { //定义了一个利用 recover 的 defer 函数。如果当前的 goroutine 产生了 panic,这个 defer 函数能够发现。当 recover() 返回非 nil 值,设置 b 为 true;
         if x := recover(); x != nil { 
             b = true
          }
       }()
     f() // 调用作为参数接收的函数。
     return // 返回 b 的值。由于 b 是命名返回值,无须指定 b。
    }

