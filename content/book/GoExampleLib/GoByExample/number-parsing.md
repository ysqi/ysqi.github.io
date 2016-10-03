---
book_chapter: "0152"
book_chapter_name: "数字解析"
book_name: "Go示例大全"
date: "2016-03-04T13:13:33+08:00"
description: ""
disqus_identifier: "book000201052"
slug: "number-parsing"
title: "Go示例大全-数字解析"
codeurl: "https://wide.b3log.org/playground/8d618fdfd830ff10406eff81d2b4d8a5.go"
---
 
从字符串中解析数字在很多程序中是一个基础常见的任务，在Go 中是这样处理的。



内置的 `strconv` 包提供了数字解析功能。



使用 `ParseFloat` 解析浮点数，这里的 `64` 表示表示解析的数的位数。

在使用 `ParseInt` 解析整形数时，例子中的参数 `0` 表示自动推断字符串所表示的数字的进制。`64` 表示返回的整形数是以 64 位存储的。

`ParseInt` 会自动识别出十六进制数。

`ParseUint` 也是可用的。

`Atoi` 是一个基础的 10 进制整型数转换函数。

在输入错误时，解析函数会返回一个错误。
 

```go
package main  
 
import "strconv"
import "fmt"  
 func main() {  
 
    f, _ := strconv.ParseFloat("1.234", 64)
    fmt.Println(f)  
 
    i, _ := strconv.ParseInt("123", 0, 64)
    fmt.Println(i)  
 
    d, _ := strconv.ParseInt("0x1c8", 0, 64)
    fmt.Println(d)  
 
    u, _ := strconv.ParseUint("789", 0, 64)
    fmt.Println(u)  
 
    k, _ := strconv.Atoi("135")
    fmt.Println(k)  
 
    _, e := strconv.Atoi("wat")
    fmt.Println(e)
}  
```
