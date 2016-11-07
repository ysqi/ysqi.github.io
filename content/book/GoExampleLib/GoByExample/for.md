---
book_chapter: "0105"
book_chapter_name: "For循环"
book_name: "Go示例大全"
date: "2016-03-04T13:13:15+08:00"
description: ""
disqus_identifier: "book00020105"
slug: "for"
title: "Go示例大全-For循环"
codeurl: "https://wide.b3log.org/playground/353ab4d4326488bfb63f93873e88b1e0.go"
---
 
`for` 是 Go 中唯一的循环结构。这里有 `for` 循环的三个基本使用方式。







最常用的方式，带单个循环条件。

经典的初始化/条件/后续形式 `for` 循环。

不带条件的 `for` 循环将一直执行，直到在循环体内使用了 `break` 或者 `return` 来跳出循环。
 

```Go
package main  
import "fmt"  
 func main() {  
 
    i := 1
    for i <= 3 {
        fmt.Println(i)
        i = i + 1
    }  
 
    for j := 7; j <= 9; j++ {
        fmt.Println(j)
    }  
 
    for {
        fmt.Println("loop")
        break
    }
}  
```
