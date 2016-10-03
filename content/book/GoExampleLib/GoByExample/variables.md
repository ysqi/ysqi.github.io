---
book_chapter: "0103"
book_chapter_name: "变量"
book_name: "Go示例大全"
date: "2016-03-04T13:13:15+08:00"
description: ""
disqus_identifier: "book00020103"
slug: "variables"
title: "Go示例大全-变量"
codeurl: "https://wide.b3log.org/playground/f57f1c3d158af4618cb3f9c0102bf03a.go"
---
 
在 Go 中，_变量_ 被显式声明，并被编译器所用来检查函数调用时的类型正确性







`var` 声明 1 个或者多个变量。

你可以申明一次性声明多个变量。

Go 将自动推断已经初始化的变量类型。

声明变量且没有给出对应的初始值时，变量将会初始化为_零值_ 。例如，一个 `int` 的零值是 `0`。

`:=` 语句是申明并初始化变量的简写，例如这个例子中的 `var f string = "short"`。
 

```go
package main  
import "fmt"  
 func main() {  
 
    var a string = "initial"
    fmt.Println(a)  
 
    var b, c int = 1, 2
    fmt.Println(b, c)  
 
    var d = true
    fmt.Println(d)  
 
    var e int
    fmt.Println(e)  
 
    f := "short"
    fmt.Println(f)
}  
```
