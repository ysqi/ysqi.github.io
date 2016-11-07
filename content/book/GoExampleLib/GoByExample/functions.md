---
book_chapter: "0112"
book_chapter_name: "函数"
book_name: "Go示例大全"
date: "2016-03-04T13:13:18+08:00"
description: ""
disqus_identifier: "book000201012"
slug: "functions"
title: "Go示例大全-函数"
codeurl: "https://wide.b3log.org/playground/8c8554aa4cbc7e68f1930e41acc2cd9c.go"
---
 
_函数_ 是 Go 的中心。我们将通过一些不同的例子来进行学习。





这里是一个函数，接受两个 `int` 并且以 `int` 返回它们的和

Go 需要明确的返回值，例如，它不会自动返回最后一个表达式的值



正如你期望的那样，通过 `name(args)` 来调用一个函数，
 

```Go
package main  
import "fmt"  
 
func plus(a int, b int) int {  
 
    return a + b
}  
 func main() {  
 
    res := plus(1, 2)
    fmt.Println("1+2 =", res)
}  
```
