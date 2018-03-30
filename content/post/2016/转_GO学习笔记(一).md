
---
date: 2016-12-31T11:33:04+08:00
title: "GO学习笔记(一)"
description: ""
disqus_identifier: 1485833584491228366
slug: "GOxue-xi-bi-ji-(yi-)"
source: "https://segmentfault.com/a/1190000006897440"
tags: 
- 变量 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

### 变量

#### 变量声明

通过关键字var声明变量，数据类型在变量名后。

    var a int
    var b int = 10

变量声明语句可以不需要使用分号作为结束符。

可以将若干个需要声明的变量放置在一起，避免重复写var关键字。

    var (
         a int
         b string
    )

#### 初始化变量

初始化变量时可以不用声明数据类型，go编译器会根据表达式右值推导出该声明为哪种类型。

    var a int = 10
    var a = 10

可以使用 :=
操作符，用于明确表达同时进行变量声明和初始化工作，可以减少写关键字var跟数据类型。

#### 变量赋值

go支持`多重赋值`

    var i
    var j
    i, j = j, i

这样就可以减少引入一个中间变量。

    var t = i;
    var i = j;
    var j = t;

#### 匿名变量

在函数返回多个值，但只希望获取个别返回值的情况下，可以使用匿名变量，这样就可以省去定义没用的变量。

    func GetName() (firstName, lastName, nickName string) {
        return "May", "Chan", "Chibi Maruko"
    }

取nickName的值，可以写成下面的写法

    _, _, nickName := GetName()

使用匿名变量的好处是可以大幅降低沟通的复杂度和代码维护的难度。

### 常量

常量是在编译期间就已知且不能改变的值。（整形、浮点型、复数型）

在Go中，以`大写字母开头`的常量在`包外可见`。

#### 字面常量

go中的字面常量是无类型的，只要该常量在相应类型的值域范围内，就可以作为该类型的常量。

#### 定义常量

使用`const`关键字声明常量，定义常量的时候可以限定常量类型，但是非必需的，如果定义时没有指定类型，那么该常量与字面常量一样是无类型的。

    const Pi float64 = 3.14159265358979323846
    const zero = 0.0
    const (
        size int64 = 1024
        eof = -1
    )
    const a, b, c = 1, 2, "hello"

常量定义表达式的右边的值可以是一个在编译期的常量表达式，如：

    const Mask = 1 << 3

但由于常量的赋值是一个编译期行为，所以右值不能出现任何需要运行期才能得出结果的表达式，如：

    const Home = os.GetEnv("HOME")

#### 预定义常量

go内置的常量有`true`、`false`、`iota`;

`iota`是可以被编译器修改的常量，在每一个const关键字出现时会被重置为0，然后在下一个`const`出现之前，没出现一次`iota`，其值都会自增1。

    const (
        c0 = iota      // c0 = 0
        c1 = iota      // c0 = 1
        c2 = iota      // c0 = 2
    )

如果两个以上`const`的赋值语句的表达式是一样的，那么刻意省略从第二个开始的赋值表达式，因为，上面的代码可以简写为：

    const (
        c0 = iota      // c0 = 0
        c1             // c0 = 1
        c2             // c0 = 2
    )

