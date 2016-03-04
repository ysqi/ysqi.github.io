---
book_chapter: "0114"
book_chapter_name: "变参函数"
book_name: "Go示例大全"
date: "2016-03-04T13:13:19+08:00"
description: ""
disqus_identifier: "book000201014"
slug: "variadic-functions"
title: "Go示例大全-变参函数"
codeurl: "https://wide.b3log.org/playground/7224c5b1199cbc1f6999512ef2bd3997.go"
---
 
[_可变参数函数_](http://zh.wikipedia.org/wiki/可變參數函數)。可以用任意数量的参数调用。例如，`fmt.Println` 是一个常见的变参函数。





这个函数使用任意数目的 `int` 作为参数。



变参函数使用常规的调用方式，除了参数比较特殊。

如果你的 slice 已经有了多个值，想把它们作为变参使用，你要这样调用 `func(slice...)`。
 

```go
package main  
import "fmt"  
 
func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}  
 func main() {  
 
    sum(1, 2)
    sum(1, 2, 3)  
 
    nums := []int{1, 2, 3, 4}
    sum(nums...)
}  
```
