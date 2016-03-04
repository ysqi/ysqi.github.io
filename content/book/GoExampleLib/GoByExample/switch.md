---
book_chapter: "0107"
book_chapter_name: "分支结构"
book_name: "Go示例大全"
date: "2016-03-04T13:13:16+08:00"
description: ""
disqus_identifier: "book00020107"
slug: "switch"
title: "Go示例大全-分支结构"
codeurl: "https://wide.b3log.org/playground/cbe5613ee8ab8a921b9aedfd9d93df2a.go"
---
 
_switch_ ，方便的条件分支语句。







一个基本的 `switch`。

在一个 `case` 语句中，你可以使用逗号来分隔多个表达式。在这个例子中，我们很好的使用了可选的`default` 分支。

不带表达式的 `switch` 是实现 if/else 逻辑的另一种方式。这里展示了 `case` 表达式是如何使用非常量的。
 

```go
package main  
import "fmt"
import "time"  
 func main() {  
 
    i := 2
    fmt.Print("write ", i, " as ")
    switch i {
    case 1:
        fmt.Println("one")
    case 2:
        fmt.Println("two")
    case 3:
        fmt.Println("three")
    }  
 
    switch time.Now().Weekday() {
    case time.Saturday, time.Sunday:
        fmt.Println("it's the weekend")
    default:
        fmt.Println("it's a weekday")
    }  
 
    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("it's before noon")
    default:
        fmt.Println("it's after noon")
    }
}  
```
