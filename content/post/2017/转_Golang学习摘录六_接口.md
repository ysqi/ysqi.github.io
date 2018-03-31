
---
date: 2017-02-24T08:31:54+08:00
title: "Golang学习摘录六:接口"
description: ""
disqus_identifier: 1487896314377836477
slug: "Golangxue-xi-zhai-lu-liu-:jie-kou"
source: "http://www.jianshu.com/p/996e840629e7"
tags: 
- golang 
categories:
- 编程语言与开发
---

Go中关键字interface被赋予了很多不同的含义。每个类型都有接口，意味着对那个类型定义了方法集合。

    // 这段代码定义了具有一个字段和两个方法的结构类型s。
    type S struct { i int }
    func (p *S) Get() int { return p.i }
    func (p *S) Put(v int) { p.i = v }
    // 定义接口
    type I interface {
      Get() int
      Put(int)
    }
    // 对于接口I，S是合法的实现，因为它定义了 I 所需的两个方法。注意：即便是没有明 确定义 S 实现了 I，这也是正确的。
    // Go 程序的特性接口值：
    func f(p I) {// 定义一个函数接受一个接口类型作为参数
      fmt.Println(p.Get()) // p实现了接口I，必须有Get()方法
      p.Put(1) // Put()方法是类似的
      // 这里的变量p保存了接口类型的值。 
    }
    // 调用
    var s S
    f(&s) // 因为S实现了I，可以调用f向其传递S类型的值的指针
    // 获取 s 的地址,而不是 S 的值的原因,是因为在 s 的指针上定义了方法,参阅上面的 代码 5.1。这并不是必须的——可以定义让方法接受值——但是这样的话 Put 方法就不 会像期望的那样工作了。

    // 定义另外一个类型同样实现接口I：
    type R struct { i int }
    func (p *R) Get() int { return p.i } 
    func (p *R) Put(v int) { p.i=v }

    // 函数f现在可以接受类型为R或S的变量。
    // 假设需要在函数f中知道实际的类型。在Go中可以使用type switch得到。
    func f(p I) {
      switch t : p.(type) { // 类型判断；在switch语句中使用(type)。保存类型到变量t中。在switch之外使用(type)是非法的。
        case *S:
        case *R:
        case S:
        case R:
        defaut:
      }
    }
    // 类型判断不是唯一的运行时得到类型的方法。 为了在运行时得到类型,同样可以使用 “comma, ok” 来判断一个接口类型是否实现了 某个特定接口:
        if t, ok := p.(*S); ok {
            fmt.Printf("%d\n", t.Get())
        }
    // 确定一个变量实现了某个接口，可以使用：
    t := p.(*S)

#### 空接口

    interface{} // 每个类型都能匹配到空接口
    func g(something interface{}) int { // 用空接口作为参数的函数
      return something.(I).Get() // 值 something 具有 类型 interface{},这意味着方法没有任何约束:它能包含任何类型。.(I) 是 类型 断言,用于转换 something 到 I 类型的接口。
    // 如果有这个类型,则可以调用 Get() 函 数。因此,如果创建一个 *S 类型的新变量,也可以调用 g(),因为 *S 同样实现了空 接口。
    }
    s = new(S)
    fmt.Println(g(s)) // 打印0
    // 如果调用g()的参数没有实现I，会在运行时得到警告，如
    i:=5 
    fmt.Println(g(i)) // 在运行时警告：panic: interface conversion: int is not main.I: missing method Get，这是没有问题的，因为内建类型int没有Get()方法。

方法
----

方法就是有接受者的函数，可以在任意类型上定义方法（除了非本地类型，包括内建类型：int类型不能有方法）。但是可以新建一个拥有方法的整数类型。

    type Foo int
    func (f Foo) Emit(){
      fmt.Printf("%v",f)
    }
    type Emitter interface{
      Emit()
    }
    func (i int) Emit() {....}  // ❎不能直接扩展内建类型
    func (a *net.AddrError) Emit(){...}// ❎不能直接扩展非本地类型
    // 可以通过新建类型的方式处理
    type newError net.AddrError
    func (a *newError) Emit(){...}// ✅

接口定义为一个方法的集合。方法包含实际的代码。换句话说，一个接口就是定义，而方法就是实现。因此，接受者不定定义为接口类型，这样做会引起invalid
receiver type编译错误

> 接收者类型必须是 T 或 \*T,这里的 T 是类型名。T 叫做接收者基础类型或
> 简称基础类型。基础类型一定不能使指针或接口类型,并且定义在与方法
> 相同的包中。
>
> 在Go中创建指向接口的指针是无意义的。实际上创建接口值的指针也是非法的。

接口名字
--------

根据规则,单方法接口命名为方法名加上 -er 后缀:Reader,Writer,Formatter
等。\
有一堆这样的命名,高效的反映了它们职责和包含的函数名。Read,Write,Close,
Flush,String 等等有着规范的声明和含义。为了避免混淆,除非有类似的声明和含
义,否则不要让方法与这些重名。相反的,如果类型实现了与众所周知的类型相同的方
法,那么就用相同的名字和声明;将字符串转换方法命名为 String 而不是
ToString。\
冒泡排序整型数据

    func bubblesort(n []int) {
      for i:=0; i< len(n)-1; i++ {
          for j:=i+1; j< len(n); j++ {
             if n[j]<n[i] {
                n[i], n[j] = n[j], n[i]
            } 
        }
      }
    }

类似的排序字符串\
func bubblesortString(n []string){ /*....*/ }\
基于此，可能需要两个函数，每个类型一个。而通过使用接口可以让这个变得更加通用。\
下面的方法不能用❎

    func sort(i []interface{}) { // 函数将接受一个空接口的slice
      switch i.(type) {// 使用type switch找到输入参数实际的类型
        case string: // 排序
            // ....
        case int:
            // ...
      }
      return /*....*/    //返回排序后的slice
    }

但是如果用sort([]int{1,4,5})调用这个函数，会失败：cannot use i (type
[]int) as type []interface in function argment\
这是因为Go不能（隐式）转换为slice。\
所以需要通过接口的方式创建Go形式的“通用”函数。\
步骤：\
1.定义一个有着若干排序相关方法的接口函数（这里叫做Sorter）。至少需要获取slice长度的函数，比较两个值的函数和交换函数；

    type Sorter interface {
      Len() int // len()作为方法
      Less(i,j int) bool // p[j]<p[i]作为方法
      Swap(i,j int)  // p[i],p[i] = p[j],p[i]作为方法
    }

2.定义用于排序slice的新类型。注意定义的是slice类型；

    type Xi []int
    type Xs []string

3.实现Sorter接口的方法。

    //整数为：
    func (p Xi) Len() int { return len(p) }
    func (p Xi) Less(i int, j int) bool { return p[j]<p[i] }
    func (p Xi) Swap(i int, j int) { p[i],p[j]=p[j],p[i] }
    // 字符串的：
    func (p Xs) Len() int { return len(p) }
     func (p Xs) Less(i int, j int) bool { return p[j] < p[i] }
     func (p Xs) Swap(i int, j int) { p[i], p[j] = p[j], p[i]

4.编写作用于Sorter接口的通用排序函数。

    func Sort(x Sorter) {// x现在是Sorter类型;
       for i:=0; i<x.Len()-1; i++ {
        for j:=i+1; j<x.Len(); j++ {
           if x.Less(i, j) {
              x.Swap(i, j)
           }
        }
      }
    }
    // 现在可以像下面这样使用通用的 Sort 函数:
    ints := Xi{44, 67, 3, 17, 89, 10, 73, 9, 14, 8}
    strings := Xs{"nut", "ape", "elephant", "zoo", "go"}

    Sort(ints)
    fmt.Printf("%v\n", ints)
    Sort(strings)
    fmt.Printf("%v\n", strings)

