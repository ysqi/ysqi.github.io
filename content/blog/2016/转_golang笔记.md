
---
date: 2016-12-31T11:32:28+08:00
title: "golang笔记"
description: ""
disqus_identifier: 1485833548644940169
slug: "golang-bi-ji"
source: "https://segmentfault.com/a/1190000008156232"
tags: 
- golang 
- emacs 
topics:
- 编程语言与开发
---

emacs 开发环境
--------------

spacemacs已经集成了不少功能，但是缺少代码提示。因此还需要[gocode](https://github.com/nsf/gocode)来辅助。

1.  安装所需的命令

    go get -u -v github.com/nsf/gocode\
    go get -u -v github.com/rogpeppe/godef go get -u -v\
    golang.org/x/tools/cmd/guru go get -u -v\
    golang.org/x/tools/cmd/gorename go get -u -v\
    golang.org/x/tools/cmd/goimports

2.  配置文件

    将gocode中的`go-autocomplete.el`拷贝至`elpa/go-mode`下\
    向space
    macs手动添加包：`dotspacemacs-additional-packages '(go-autocomplete)`\
    在`dotspacemace/user-config`中添加以下内容

        (require 'go-autocomplete)
        (require 'auto-complete-config)
        (ac-config-default)

------------------------------------------------------------------------

GO
==

[package 手册](http://docs.studygolang.com/pkg/)
、[go语言圣经](https://github.com/gopl-zh/gopl-zh.github.com)

语法
----

1.  声明 ：`var|const|type|func name (类型) (值)`

    `var` 显式声明一个变元。`var name Type`\
    `:=` 语法可以隐式地声明一个变元。`name := f() | a`\
    隐式声明的变元的作用域是可以被覆盖的，但显式的不能。

        var a int //1.会导致3出错
        a := 1    //2.不会导致3处出错
        {
        a:= 2     //3.
        }

    `const a=2`, `const a float64=2`

    `type name define`，`define= Type | struct{..}|interface{}`

    `func (name Type)* name (arg...) {...}`\
    打\*号的部分是
    接收器(receiver)，用于扩展指定`Type`（必须是自定义类型）。

        type Point struct{ X, Y float64 }
        // traditional function
        func (p Point) Distance(q Point) float64 {
            return math.Hypot(q.X-p.X, q.Y-p.Y)
        }
        type Float64 float64 //
        func (p Float64) Distance(q Float64) float64 {
            return math.Abs(float64(p - q))
        }
        var x, y Float64 = 1.0, 2
        x.Distance(y)

    匿名函数：`func(r rune) rune { return r + 1 }`\
    便捷操作

        import (
            "C"
            "fmt"
            "math"
        )
        const | var (
            AbsoluteZeroC Celsius = -273.15 // 绝对零度
            FreezingC     Celsius = 0       // 结冰点温度
            BoilingC      Celsius = 100     // 沸水温度
        )
        var a,b,c Type

2.  控制结构\
    **if**语句

        if a, b := 21, 3; a > b {
            fmt.Println("a>b ? true")
        }else {
        }

        for i, j := 1, 10; i < j; i,j=i+1,j+1 {  //死循环
            fmt.Println(i)
        }

    **switch**语句

        switch ch {
        case '0': 
            fallthrough   //必须是最后一个语句
        case '1':
            cl = "Int"
        case 'A': 
        case 'a':
            fallthrough
            cl = "ABC"    //error
        default:
            cl = "Other Char"
        }

3.  类型转换\
    语法：

        <目标类型> ( <表达式> )
        <目标类型的值>，<布尔参数> := <表达式>.( 目标类型 ) // 安全类型断言
        <目标类型的值> := <表达式>.( 目标类型 )　　//非安全类型断言

        var3 := int64(var1)
        var i interface{} = "TT"
        j, b := i.(int)
        if b {
            fmt.Printf("%T->%d\n", j, j)
        } else {
            fmt.Println("类型不匹配")
        }

基本类型
--------

1.  数组

        q := [...]int{1, 2, 3}
        q := [3]int{1, 2, 3} //等价
        fmt.Printf("%T\n", q) // "[3]int"

        r := [...]int{99: -1}// {下表：值}，填充默认值

2.  Slice\
    Slice（切片）代表变长的序列，序列中每个元素都有相同的类型。一个slice类型一般写作`[]T`，其中T代表slice中元素的类型；数组和slice之间有着紧密的联系。一个slice是一个轻量级的数据结构，提供了访问数组子序列（或者全部）元素的功能，而且slice的底层确实引用一个数组对象。一个slice由三个部分构成：指针、长度和容量。\
    `array[i:j]`来创建一个Slice，0 ≤ i≤ j≤ cap(s)

        array := [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}
        front := array[0:5]
        mid := array[3:8]
        tail := array[4:9]
        front[4] = 555 //3个数都被修改
        array[5] = 605 //会导致 mid,tail 修改

    **注意**

            array := [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9}
            front := array[0:5]
            mid := array[3:8]
            tail := array[4:9]
            nf := append(tail, 66) //1.
            nf := append(mid, 66) //2.
            nf[0] = 5555
            fmt.Println(front, mid, tail, nf)
            //1. [1 2 3 4 601] [4 601 6 7 8] [601 6 7 8 9] [5555 6 7 8 9 66]
            //2. [1 2 3 5555 5] [5555 5 6 7 8] [5 6 7 8 66] [5555 5 6 7 8 66]

    当原数组够用时，append会直接使用原来的空间。不够时另开一片。

    `[]T`是切片类型 `[n]T`是数组类型

3.  Map\
    我们也可以用map字面值的语法创建map，同时还可以指定一些最初的key/value：

        ages := make(map[string]int) // mapping from strings to ints
        ages := map[string]int{
            "alice":   31,
            "charlie": 34,
        }

4.  结构体

        type Employee struct {
            ID        int
            Name      string
            Address   string
            DoB       time.Time
            Position  string
            Salary    int
            ManagerID int
        }

        var dilbert Employee

5.  接口\
    一类有相同方法的对象。

        package io
        type Reader interface {
            Read(p []byte) (n int, err error)
        }
        type Closer interface {
            Close() error
        }
        type ReadWriter interface {
            Read(p []byte) (n int, err error)
            Writer
        }

        var w io.Writer
        w = os.Stdout //使用接口

符号
----

    关键字：
            break      default       func     interface   select
            case       defer         go       map         struct
            chan       else          goto     package     switch
            const      fallthrough   if       range       type
            continue   for           import   return      var
                
    内建常量: true false iota nil

    内建类型: int int8 int16 int32 int64
              uint uint8 uint16 uint32 uint64 uintptr
              float32 float64 complex128 complex64
              bool byte rune string error

    内建函数: make len cap new append copy close delete
              complex real imag
              panic recover

1.  defer ：延迟执行

            defer println("p1") //后被打印
            defer println("p2") //先被打印

    假设defer处声明了一个变量，那么在析构的时候执行内容。

2.  range ：配合for使用

        for index, value := range mySlice {
            fmt.Println("index: " + index)
            fmt.Println("value: " + value)
        }

    -   for index,char := range string {}

    -   for index,value := range array {}

    -   for index,value := range slice {}

    -   for key,value := range map {}

抽象
----

不同于面向对象的语言先定义一个对象具有哪些抽象行为再实现的思路。\
GO是先实现一个对象，再检查这个对象是否符合接口规范。

1.  方法\
    扩展某一个类型

        type Point struct{ X, Y float64 }

        // traditional function
        func Distance(p, q Point) float64 {
            return math.Hypot(q.X-p.X, q.Y-p.Y)
        }

        // same thing, but as a method of the Point type
        func (p Point) Distance(q Point) float64 {
            return math.Hypot(q.X-p.X, q.Y-p.Y)
        }

2.  接口\
    一类有相同方法的对象。

        package io
        type Reader interface {
            Read(p []byte) (n int, err error)
        }
        type Closer interface {
            Close() error
        }
        type ReadWriter interface {
            Read(p []byte) (n int, err error)
            Writer
        }

并行
----

1.  goroutine\
    使用go关键字go

        f()    // call f(); wait for it to return
        go f() // create a new goroutine that calls f(); don't wait

2.  channel

        var ch chan int
        ch = make(chan int)    // unbuffered channel
        ch = make(chan int, 0) // unbuffered channel
        ch = make(chan int, 3) // buffered channel with capacity 3

        ch <- 2 //send
        a := ch //receive

    `chan<- int`表示一个只发送int的channel，只能发送不能接收。\
    `<-chan int`表示一个只接收int的channel，只能接收不能发送。\
    一个基于无缓存Channels的发送操作将导致发送者goroutine阻塞，直到另一个goroutine在相同的Channels上执行接收操作，当发送的值通过Channels成功传输之后，两个goroutine可以继续执行后面的语句。

3.  select

        select {
        case <-ch1:
            // ...
        case x := <-ch2:
            // ...use x...
        case ch3 <- y:
            // ...
        case <-time.After(10 * time.Second):
            //超时机制，不能与default一起
        default:
            // ...
        }

调用C库
-------

[cgo](https://golang.org/cmd/cgo/)

    /*
    #cgo CFLAGS: -I/usr/include
    #cgo LDFLAGS: -L/usr/lib -lbz2
    #include <bzlib.h>
    #include <stdlib.h>
    bz_stream* bz2alloc() { return calloc(1, sizeof(bz_stream)); }
    int bz2compress(bz_stream *s, int action,
                    char *in, unsigned *inlen, char *out, unsigned *outlen);
    void bz2free(bz_stream* s) { free(s); }
    */
    import "C"

可以添加一下编译选项：

> CFLAGS, CPPFLAGS, CXXFLAGS, FFLAGS , LDFLAGS

c代码必须加以注释且后面紧跟`import "C"`

在Go中使用C的类型

> C.char C.schar (signed char) C.uchar (unsigned char) C.short C.ushort
> (unsigned short) C.int C.uint (unsigned int) C.long C.ulong (unsigned
> long) C.longlong (long long) C.ulonglong (unsigned long long) C.float
> C.double C.complexfloat (complex float) C.complexdouble (complex
> double)

如果是 `struct, union, or enum` 类型的，会添加前缀
`struct_, union_, or enum_`。\
如`struct。a{...};`在go中变为`C.struct_a`。\
`C.sizeof_T` 表示C中某种类型的长度

    package main

    // typedef int (*intFunc) ();
    //
    // int
    // bridge_int_func(intFunc f)
    // {
    //        return f();
    // }
    //
    // int fortytwo()
    // {
    //        return 42;
    // }
    import "C"
    import "fmt"

    func main() {
        f := C.intFunc(C.fortytwo)
        fmt.Println(int(C.bridge_int_func(f)))
        // Output: 42
    }

在go中不能调用c中**可变参数的函数**。

