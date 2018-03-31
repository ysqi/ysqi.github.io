
---
date: 2016-12-31T11:34:33+08:00
title: "Go语言的标识符、关键字、字面量、类型"
description: ""
disqus_identifier: 1485833673475146628
slug: "Goyu-yan-de-biao-shi-fu-、guan-jian-zi-、zi-mian-liang-、lei-xing"
source: "https://segmentfault.com/a/1190000002687627"
tags: 
- 数据类型 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

一直在 Segment Fault
上面实行自己的拿来主义，但其实我是一直很乐意分享的人，而且特别喜欢写，以前一直都是在自己的博客里面写，但是没啥人看，也形成不了交流，所以，申请在
Segment Fault 上面开个专栏，以后还忘大家多多指教，这篇文章只是想试试
Segment Fault的编辑器，内容是前几天写的。

Go语言的语言符号又称记法元素，共包括5类，标签符（`identifier`）、关键字（`keyword`）、操作符（`operator`）、分隔符（`delimiter`）、字面量（`literal`），它们是组成Go语言代码和程序的最基本单位。

Go语言的所有源代码都必须由 `Unicode` 编码规范的 UTF-8 编码格式进行编码。

### 一、标识符

在Go语言代码中，每一个标识符可以代表一个变更或者一个类型（即标识符可以被看作是变量或者类型的代号或者名称），标识符是由若干字母、下划线（\_）和数字组成的字符序列，第一个字符必须为字母。同时，使用一个标识符在使用前都必须先声明。在一个代码块中，不允许重复声明同一个标识。

> 代码包声明（`package PKG_NAME`）并不算是一个声明，因为代码包名称并不出现在任何一个作用域中，代码包声明语句的目的只是为了鉴别若干源码文件是否属于同一个代码包，或者指定导入代码包时的默认代码包引用名称。
>
> 一个限定标识符代表了对另一个代码包中的某个标识符的访问，这需要两个条件：
>
> 1.  另一个代码包必须事情由Go语言的导入声明 `import` 导入；
> 2.  某个标识符在代码包中是可导出的。
>
> 标识符可导出的前提条件有下面这两个：
>
> 1.  标识符名称中的第一个字符必须是大写；
> 2.  标识必须是被声明在一个代码包中的变量或者类型的名称，或者是属于某个结构体类型的字段名称或者方法名称。

在Go语言中还存在一类特殊的标识符，叫作预定义标识符，这类标识符随Go语言的源码一同出现，主要包括以下几种：

1.  所有基本数据类型的名称
2.  接口类型 `error`
3.  常量 `true`、`false` 以及 `iota`
4.  所有内奸函数的名称，即
    `append`、`cap`、`close`、`complex`、`copy`、`delete`、`imag`、`len`、`make`、`new`、`panic`、`print`、`println`、`real`和
    `recover`

由一个下划线表示的标识 \_
为空标识符，它一般被用在一需要引入一个新绑定声明中，如：

    import _ "runtime/cgo"

### 二、关键字

关键字是指被编程语言保留页不让编程人员作为标识符使用的字符序列。因此，关键字也称为保留字。

Go 语言中所有的关键只有25个：

1.  程序声明：`import`、`package`
2.  程序实体声明和定义：`chan`、`const`、`func`、`interface`、`map`、`struct`、`type`、`var`
3.  程序流程控制：`go`、`select`、`break`、`case`、`continue`、`default`、`defer`、`else`、`fallthrough`、`for`、`goto`、`if`、`range`、`return`、`switch`

### 三、字面量

简单来说，字面量就是表示值的一种标记法，但是在Go语言中，字面量的含义要更广一些：

1.  用于表示基础数据类型值的各种字面量。
2.  用户构造各种自定义的复合数据类型的类型字面量，如下面这个字面量表示了一个名称为
    Person 的自定义结构体类型：

        type Person struct {
             Name string
             Age uint8
             Address string
        }

3.  用于表示复合数据类型的值的复合字面量，更确切地讲，它会被用来构造类型
    Struct（结构体）、Array（数组）、Slice（切片）和Map（字典）的值。如下面的字面量可以表示上面的那个
    Person 结构体类型的值：

        Person(Name: "Eric Pan", Age: 28, Address: "Beijing China"}

### 四、类型

一个类型确定了一类值的集合，以及可以在这些值上施加的操作。类型可以由类型名称或者类型字面量指定，类型分为基本类型和复合类型，基本类型的名称可以代表其自身，比如：

    var name string 

string 即为一个基本类型，Go
语言中的基本类型有：bool、byte、int/uint、int8/uint8、int16/uint16、int32/uint32、int64/uint64、float32、float64、complex64、complex128，共18个，基本类型的名称都必须预定义标识符。除了
bool 与 string 外，其它的都称为数值类型。

除了基本类型外，Go语言还有八个复合类型：Array（数组）、Struct（结构体）、Function（函数）、Interface（接口）、Slice（切片）、Map（字典）、Channel（通道）以及Pointer（指针）。

复合类型一般由若干（包括0）个其他已被定义的类型组合而成，如定义一本书的结构体：

    type Book struct {
         Name string
         ISBN string
         Press string
         TotalPages uint16
    }

Go语言中的类型又可以分为静态类型和动态类型，一个变量的静态类型是指在变量声明中示出的那个类型，绝大多数类型的变量都只拥有静态类型，唯独接口类型的变量例外，它除了拥有静态类型之外，还拥有动态类型，这个动态类型代表了在运行时与该变量绑定在一起的值的实际类型。

> 每一个类型都会有一个潜在类型，如果这个类型是一个预定义的类型（也就是基本类型），或者是一个由类型字面量构造的复合类型，那么它的潜在类型就是它自身，比如
> string 类型的潜在类型就是 string，在上面提到的 Book 的潜在类型就是
> Book，但是如果一个类型并不属于上述情况，那么这个类型的潜在类型就是在类型声明中的那个类型的潜在类型，比如我们按以下方式声明一个
> MyString 类型：

    type MyString string

MyString 类型的潜在类型就是 string 类型的潜在类型，实际上，我们可以将
MyString 看作是 string 类型的一个别名，在Go 语言中的基本数据类型 rune
类型就是如此，它可以看作是 uint32 类型的一个别名类型，其潜在类型就是
uint32 ，但是一类要注意， MyString 与 string 却并不是一个相同的类型。

潜在类型在声明过程中是具有可传递性的，如下面我们再声明一个 iString
类型：

    type iString MyString

iString 类型的潜在类型同样就是 string 类型。

