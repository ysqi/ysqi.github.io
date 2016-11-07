---
book_chapter: "0139"
book_chapter_name: "排序"
book_name: "Go示例大全"
date: "2016-03-04T13:13:28+08:00"
description: ""
disqus_identifier: "book000201039"
slug: "sorting"
title: "Go示例大全-排序"
codeurl: "https://wide.b3log.org/playground/b8638a022e56631ff3649996e181bf0b.go"
---
 
Go 的 `sort` 包实现了内置和用户自定义数据类型的排序功能。我们首先关注内置数据类型的排序。







排序方法是正对内置数据类型的；这里是一个字符串的例子。注意排序是原地更新的，所以他会改变给定的序列并且不返回一个新值。

一个 `int` 排序的例子。

我们也可以使用 `sort` 来检查一个序列是不是已经是排好序的。
 

```Go
package main  
import "fmt"
import "sort"  
 func main() {  
 
    strs := []string{"c", "a", "b"}
    sort.Strings(strs)
    fmt.Println("Strings:", strs)  
 
    ints := []int{7, 2, 4}
    sort.Ints(ints)
    fmt.Println("Ints:   ", ints)  
 
    s := sort.IntsAreSorted(ints)
    fmt.Println("Sorted: ", s)
}  
```
