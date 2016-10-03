---
book_chapter: "0106"
book_chapter_name: "if/else 分支"
book_name: "Go示例大全"
date: "2016-03-04T13:13:16+08:00"
description: ""
disqus_identifier: "book00020106"
slug: "if-else"
title: "Go示例大全-if/else 分支"
codeurl: "https://wide.b3log.org/playground/5773b3bef48202239d7bb55a892cef8c.go"
---
 
`if` 和 `else` 分支结构在 Go 中当然是直接了当的了。







这里是一个基本的例子。

你可以不要 `else` 只用 `if` 语句。

在条件语句之前可以有一个语句；任何在这里声明的变量都可以在所有的条件分支中使用。

注意，在 Go 中，你可以不适用圆括号，但是花括号是需要的。
 

```go
package main  
import "fmt"  
 func main() {  
 
    if 7%2 == 0 {
        fmt.Println("7 is even")
    } else {
        fmt.Println("7 is odd")
    }  
 
    if 8%4 == 0 {
        fmt.Println("8 is divisible by 4")
    }  
 
    if num := 9; num < 0 {
        fmt.Println(num, "is negative")
    } else if num < 10 {
        fmt.Println(num, "has 1 digit")
    } else {
        fmt.Println(num, "has multiple digits")
    }
}  
 
```
