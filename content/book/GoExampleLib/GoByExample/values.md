---
book_chapter: "0102"
book_chapter_name: "值"
book_name: "Go示例大全"
date: "2016-03-04T13:13:14+08:00"
description: ""
disqus_identifier: "book00020102"
slug: "values"
title: "Go示例大全-值"
codeurl: "https://wide.b3log.org/playground/4a18aa883b57c679e42e092f5bc418a2.go"
---
 
Go 拥有各值类型，包括字符串，整形，浮点型，布尔型等。下面是一些基本的例子。







字符串可以通过 `+` 连接。

整数和浮点数

布尔型，还有你想要的逻辑运算符。
 

```go
package main  
import "fmt"  
 func main() {  
 
    fmt.Println("go" + "lang")  
 
    fmt.Println("1+1 =", 1+1)
    fmt.Println("7.0/3.0 =", 7.0/3.0)  
 
    fmt.Println(true && false)
    fmt.Println(true || false)
    fmt.Println(!true)
}  
```
