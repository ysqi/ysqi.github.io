---
date: 2013-04-03
title: 老虞学golang-变量声明与初始化
slug: ysqi-golang-var-init
categories:
- 编程语言与开发
tags:
- Golang
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100013
---

### 变量声明
>官方doc: http://golang.org//spec#Variable_declarations

Go中使用全新的关键字var来声明变量。var我们并不陌生，在Javascript 和C#中均有出现。不同的是Go和C#中变量属于强类型，在声明变量后就不允许改变其数据类型。


**声明变量有多种形态:**
```Go
var a int //声明一个int类型的变量

var b struct { //声明一个结构体
    name string
}

var a = 8 //声明变量的同时赋值，编译器自动推导其数据类型
var a int = 8 //声明变量的同时赋值

var { //批量声明变量，简洁
    a int
    b string
}
```

### 变量初始化

变量的初始化工作可以在声明变量时进行初始化，也可以先声明后初始化。此时var关键字不再是必须的。

初始化变量有多种方式，每种方式有不同的使用场景：

在方法中声明一个临时变量并赋初值

```Go
var tmpStr = “”
var tmpStr string = “”
tmpStr :=””
```

全局中已声明变量直接赋值

```Go
tmpStr = “<body>”
```

我们看到有此两种方式：

`var name [type] = value`

如果不书写 type ,则在编译时会根据value自动推导其类型。

`name := value`

这里省略了关键字var，我很喜欢这种方式（可以少写代码，而没有任何坏处）。 但这有需要注意的是“ :=” 是在声明和初始化变量，因此该变量必须是第一次出现，如下初始化是错误的。但是要注意赋值时要确定你想要的类型，在Go中不支持隐式转换的。如果是定义个float64类型的变量，请写为 `v1 :=8.0` 而不是`v1 :=8` 。
```Go
var a int

a := 8
```

**NODE:**我刚开始时老出现这种错误，一直将 “:= ” 当作 一般赋值语句处理


### 思考问题

初始化语句，在编译器上是如何进行自动类型推导的。一些字面常量是如何归类的？

如 8 → int , 8.0 → float64
```Go
package main

import (
	"fmt"
	"reflect"
)

func main() {

	var v1 int = 8

	var v2 byte = 8

	v3 := 8

	v4 := 8.0

	fmt.Printf("v1 type :%s\n", reflect.TypeOf(v1)) //int

	fmt.Printf("v2 type :%s\n", reflect.TypeOf(v2)) //uint8

	fmt.Printf("v3 type :%s\n", reflect.TypeOf(v3)) //int

	fmt.Printf("v4 type :%s\n", reflect.TypeOf(v4)) //float64

}
```



官方文档： http://golang.org/ref/spec#Constant_expressions

```Go
const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
const b = 15 / 4           // b == 3     (untyped integer constant)
const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
const d = 1 << 3.0         // d == 8     (untyped integer constant)
const e = 1.0 << 3         // e == 8     (untyped integer constant)
const f = int32(1) << 33   // f == 0     (type int32)
const g = float64(2) >> 1  // illegal    (float64(2) is a typed floating-point constant)
const h = "foo" > "bar"    // h == true  (untyped boolean constant)
const j = true             // j == true  (untyped boolean constant)
const k = 'w' + 1          // k == 'x'   (untyped rune constant)
const l = "hi"             // l == "hi"  (untyped string constant)
const m = string(k)        // m == "x"   (type string)
const Σ = 1 - 0.707i       //            (untyped complex constant)
const Δ = Σ + 2.0e-4       //            (untyped complex constant)
const Φ = iota*1i - 1/1i   //            (untyped complex constant)
const ic = complex(0, c) // ic == 3.75i (untyped complex constant)

const iΘ = complex(0, Θ)   // iΘ == 1.5i   (type complex128)
const Huge = 1 << 100

// Huge == 1267650600228229401496703205376 (untyped integer constant)

const Four int8 = Huge >> 98  // Four == 4      (type int8)


^1 // untyped integer constant, equal to -2

uint8(^1)  // illegal: same as uint8(-2), -2 cannot be represented as a uint8
^uint8(1)  // typed uint8 constant, same as 0xFF ^ uint8(1) = uint8(0xFE)
int8(^1)   // same as int8(-2)
^int8(1)   // same as -1 ^ int8(1) = -2

```

------------------------2013-04-14补充----------------------------------------
```Go
func  main(){
        v1 :=8
        v1 =8.0       // 编译可通过，运行无错误
        fmt.Println(v1)
}
```
在C#中会编译不通过，而Go语言中却是可行的，这是为什么呢？我在golang-chang中向大家请教了该问题，谢谢帮助我的人。

https://groups.google.com/forum/?fromgroups=#!topic/golang-china/k1UOr_1K_yw

将结果整理给大家:

1. Go里面不损失精度的情况下会把8.0这类浮点数视作整数8
2. Go里面的常数是高精度数，分为几类：
    1. 有类型的：uint(8)，类型显式指定了，在表达式里面不会变化。
    2. 无类型的：分成无类型整数和无类型浮点两类。这两类在使用的时候会根据上下文需要的类型转化为实际类型，
    比如uint8(0) + 1.0就是uint8(1)，但是uint8(0)+1.2就会由于1.2无法转化为uint8而报错。
    如果上下文无法确定（比如 i, j := 1, 2.0这样的），那么整数无类型常数转化为int，浮点数无类型常数转化为float64.

具体规则参见：
http://tip.golang.org/ref/spec#Constant_expressions

-------------  2013-04-14 end------------------------------------------------
