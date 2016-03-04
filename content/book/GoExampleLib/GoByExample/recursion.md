---
book_chapter: "0116"
book_chapter_name: "递归"
book_name: "Go示例大全"
date: "2016-03-04T13:13:19+08:00"
description: ""
disqus_identifier: "book000201016"
slug: "recursion"
title: "Go示例大全-递归"
codeurl: "https://wide.b3log.org/playground/e1ae99a129db5b5c798e0069f49feb44.go"
---
 
Go 支持 <a href="http://zh.wikipedia.org/wiki/%E9%80%92%E5%BD%92_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)"><em>递归</em></a>。这里是一个经典的阶乘示例。





`face` 函数在到达 `face(0)` 前一直调用自身。


 

```go
package main  
import "fmt"  
 
func fact(n int) int {
    if n == 0 {
        return 1
    }
    return n * fact(n-1)
}  
 func main() {
    fmt.Println(fact(7))
}  
```
