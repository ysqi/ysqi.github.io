---
book_chapter: "0113"
book_chapter_name: "多返回值"
book_name: "Go示例大全"
date: "2016-03-04T13:13:18+08:00"
description: ""
disqus_identifier: "book000201013"
slug: "multiple-return-values"
title: "Go示例大全-多返回值"
codeurl: "https://wide.b3log.org/playground/4c1aa08bc188a90addc3d83c434e9cd1.go"
---
 
Go 内建_多返回值_ 支持。这个特性在 Go 语言中经常被用到，例如用来同时返回一个函数的结果和错误信息。





`(int, int)` 在这个函数中标志着这个函数返回 2 个 `int`。



这里我们通过_多赋值_ 操作来使用这两个不同的返回值。

如果你仅仅想返回值的一部分的话，你可以使用空白定义符 `_`。
 

```go
package main  
import "fmt"  
 
func vals() (int, int) {
    return 3, 7
}  
 func main() {  
 
    a, b := vals()
    fmt.Println(a)
    fmt.Println(b)  
 
    _, c := vals()
    fmt.Println(c)
}  
```
