---
book_chapter: "0108"
book_chapter_name: "数组"
book_name: "Go示例大全"
date: "2016-03-04T13:13:16+08:00"
description: ""
disqus_identifier: "book00020108"
slug: "arrays"
title: "Go示例大全-数组"
codeurl: "https://wide.b3log.org/playground/2b544cb2772d7dc22b85d6e7b281de69.go"
code_run_disable: true
---

在 Go 中，_数组_ 是一个固定长度的数列。







这里我们创建了一个数组 `a` 来存放刚好 5 个 `int`。元素的类型和长度都是数组类型的一部分。数组默认是零值的，对于 `int` 数组来说也就是 `0`。

我们可以使用 `array[index] = value` 语法来设置数组指定位置的值，或者用 `array[index]` 得到值。

使用内置函数 `len` 返回数组的长度

使用这个语法在一行内初始化一个数组

数组的存储类型是单一的，但是你可以组合这些数据来构造多维的数据结构。


```Go
package main  
import "fmt"  
 func main() {  
 
    var a [5]int
    fmt.Println("emp:", a)  
 
    a[4] = 100
    fmt.Println("set:", a)
    fmt.Println("get:", a[4])  
 
    fmt.Println("len:", len(a))  
 
    b := [5]int{1, 2, 3, 4, 5}
    fmt.Println("dcl:", b)  
 
    var twoD [2][3]int
    for i := 0; i < 2; i++ {
        for j := 0; j < 3; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
}  
```
