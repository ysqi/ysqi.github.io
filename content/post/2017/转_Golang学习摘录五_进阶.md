
---
date: 2017-02-24T08:31:54+08:00
title: "Golang学习摘录五:进阶"
description: ""
disqus_identifier: 1487896314654082011
slug: "Golangxue-xi-zhai-lu-wu-:jin-jie"
source: "http://www.jianshu.com/p/ed6c139cf9fc"
tags: 
- golang 
topics:
- 编程语言与开发
---

Go 有指针。然而却没有指针运算,因此它们更象是引用而不是你所知道的来自于 C
的指针。指针非常有用。在 Go
中调用函数的时候,得记得变量是值传递的。因此,为了修改一个传递入函数的值的效率和可能性,有了指针。

    var p *int
    fmt.Printf("%v", p) // 打印nil
    var i int // 定义一个整型变量i
    p = &i // 使得p指向i
    fmt.Printf("%v", p) // 打印出来的内容类似0x7ff96b81c000a

内存分配
--------

#### 用new分配内存

内建函数 new 本质上说跟其他语言中的同名函数功能一样:new(T)
分配了零值填充 的 T 类型的内存空间,并且返回其地址,一个 \*T 类型的值。用
Go 的术语说,它返回 了一个指针,指向新分配的类型 T
的零值。记住这点非常重要。

#### 用make分配内存

内建函数 make(T, args) 与 new(T) 有着不同的功能。它只能创建 slice,map 和
channel,并且返回一个有初始值(非零)的 T 类型,而不是 \*T。本质
来讲,导致这三个类型有所不同的原因是指向数据结构的引用在使用前必须被初始化。
例如,一个 slice,是一个包含指向数据(内部
array)的指针,长度和容量的三项描述 符;在这些项目被初始化之前,slice 为
nil。对于 slice,map 和 channel,make 初始
化了内部的数据结构,填充适当的值。

> new分配；make初始化
>
> -   new(T)返回\*T指向一个零值T
> -   make(T)返回初始化后的T\
>     make 仅适用于 map,slice 和 channel,并且返回的不是指针。应当用 new
>     获 得特定的指针。

构造函数与符合声明
------------------

    func NewFile(fd int, name string) *File { 
      if fd<0 {
        return nil
       }
      f := File{fd, name, nil, 0} // 创建一个新的File
      return &f // 返回 f 的地址 
    }

返回本地变量的地址没有问题;在函数返回后,相关的存储区域仍然存在。
事实上,从复合声明获取分配的实例的地址更好（从复合声明中获取地址,意味着告诉编译器在堆中分配空间,而不是栈中）,因此可以最终将两行缩短到一行。\
`return &File{fd, name, nil, 0}`

所有的项 目(称作 字段)都必须按顺序全部写上。然而,通过对元素用字段:
值成对的标识,初
始化内容可以按任意顺序出现,并且可以省略初始化为零值的字段。因此可以这样\
`return &File{fd: fd, name: name}`\
在特定的情况下,如果复合声明不包含任何字段,它创建特定类型的零值。表达式`new(File)`和
`&File{}` 是等价的。

复合声明同样可以用于创建 array,slice 和 map,通过指定适当的索引和 map
键来标 识字段。在这个例子中,无论是 Enone,Eio 还是 Einval
初始化都能很好的工作,只 要确保它们不同就好了。

    ar := [...] s t r i n g { Enone: "no error", Einval: "invalid argument" } 
    sl := [] s t r i n g { Enone: "no error", Einval: "invalid argument" }
    ma := map[int]string {Enone: "no error", Einval: "invalid argument"}

定义自己的类型
--------------

Go允许定义新的类型，通过关键字type实现：\
例如：

    type foo int \\ 创建了一个新的类型 foo 作用跟 int 一样
    \\ 创建更加复杂的类型需要用到 struct 关键 字。
    package main
    import "fmt"
    type NameAge struct {
     name string // 首字母小写，不导出
     age int  // 不导出
    }
    func main() {
      a := new(NameAge)
      a.name = "Pete"; a.age = 42
      fmt.Printf("%v\n", a) // 输出&{Pete 42}
    }

#### 方法

给新定义的类型创建函数有两个途径：

    // 1、创建一个函数接受这个类型的参数。
    func doSomething(n1 *NameAge, n2 int) { /* */ }
    // 2、创建一个接收方是该类型的方法
    func (n1 *NameAge) doSomething(n2 int) { /* */ }
    //方法调用方式
    var n *NameAge
    n.doSomething(2)

    // Mutex 数据类型有两个方法,Lock 和 Unlock。
    type Mutex struct { /* Mutex 字段 */ } 
    func (m *Mutex) Lock(){ /* Lock 实现 */ }
     func (m *Mutex) Unlock(){ /* Unlock 实现 */ }
    //现在用两种不同的风格创建了两个数据类型。
    type NewMutex Mutex
    type PrintableMutex struct {Mutex}
    //现在 NewMutux 等同于 Mutex,但是它没有任何 Mutex 的方法。换句话说,它的方法 是空的。但是 PrintableMutex 已经从 Mutex 继承了方法集合。*PrintableMutex 的方法集合包含了 Lock 和 Unlock 方法,被绑定到其匿名字段 Mutex。

转换
----

Go支持将一个类型转换为另一个类型

    // 1、string->byte或rune的slice
    mystring := "hello this is string"
    byteslice := []byte(mystring) // 转换成byte slice，每个byte保存字符串对应字节的整数值。注意Go的字符串是UTF-8编码，字节长度不定。
    runeslice := []rune(mystring) // 转换到rune slice，每个rune保存Unicode编码的指针。字符串中的每个字符对应一个整数。
    // 2、从字节或整型的slice到string
    b := []byte {'h','e','l','l','o'} // 复合声明 
    s := string(b)
    i := []rune {257,1024,65}
    r := string(i)
    // 3、数值的转换
    • 将整数转换到指定的(bit)长度:uint8(int);
    • 从浮点数到整数:int(float32)。这会截断浮点数的小数部分; 
    • 其他的类似:float32(int)。
    // 4、用户定义类型的转换
    type foo struct { int } // 匿名字段
     type bar foo // bar是foo的别名
    var b bar = bar{1} // 声明b为bar类型
    var f foo = b     //❎，不能使用b（类型bar）作为类型foo赋值
    var f foo = foo(b)//✅，可以通过此方式转化

组合
----

Go 不是面向对象语言,因此并没有继承。但是有时又会
需要从已经实现的类型中“继承”并修改一些方法。在 Go
中可以用嵌入一个类型的方 式来实现。

