---
book_chapter: "0104"
book_chapter_name: "常量"
book_name: "Go示例大全"
date: "2016-03-04T13:13:15+08:00"
description: ""
disqus_identifier: "book00020104"
slug: "constants"
title: "Go示例大全-常量"
codeurl: "https://wide.b3log.org/playground/2dd1d31b3e2cde5767d7bcdeb064b621.go"
---
 
Go 支持字符、字符串、布尔和数值 _常量_ 。



`const` 用于声明一个常量。



`const` 语句可以出现在任何 `var` 语句可以出现的地方

常数表达式可以执行任意精度的运算

数值型常量是没有确定的类型的，直到它们被给定了一个类型，比如说一次显示的类型转化。

当上下文需要时，一个数可以被给定一个类型，比如变量赋值或者函数调用。举个例子，这里的 `math.Sin`函数需要一个 `float64` 的参数。
 

```go
package main  
import "fmt"
import "math"  
 
const s string = "constant"  
 func main() {
    fmt.Println(s)  
 
    const n = 500000000  
 
    const d = 3e20 / n
    fmt.Println(d)  
 
    fmt.Println(int64(d))  
 
    fmt.Println(math.Sin(n))
}  
```
