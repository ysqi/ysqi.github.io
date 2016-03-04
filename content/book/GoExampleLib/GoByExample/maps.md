---
book_chapter: "0110"
book_chapter_name: "关联数组"
book_name: "Go示例大全"
date: "2016-03-04T13:13:17+08:00"
description: ""
disqus_identifier: "book000201010"
slug: "maps"
title: "Go示例大全-关联数组"
codeurl: "https://wide.b3log.org/playground/f070aee9384d92d10e2e807a3814617d.go"
---
 
_map_ 是 Go 内置[关联数据类型](http://zh.wikipedia.org/wiki/关联数组)（在一些其他的语言中称为_哈希_ 或者_字典_ ）。







要创建一个空 map，需要使用内建的 `make`:`make(map[key-type]val-type)`.

使用典型的 `make[key] = val` 语法来设置键值对。

使用例如 `Println` 来打印一个 map 将会输出所有的键值对。

使用 `name[key]` 来获取一个键的值

当对一个 map 调用内建的 `len` 时，返回的是键值对数目

内建的 `delete` 可以从一个 map 中移除键值对

当从一个 map 中取值时，可选的第二返回值指示这个键是在这个 map 中。这可以用来消除键不存在和键有零值，像 `0` 或者 `""` 而产生的歧义。

你也可以通过这个语法在同一行申明和初始化一个新的map。
 

```go
package main  
import "fmt"  
 func main() {  
 
    m := make(map[string]int)  
 
    m["k1"] = 7
    m["k2"] = 13  
 
    fmt.Println("map:", m)  
 
    v1 := m["k1"]
    fmt.Println("v1: ", v1)  
 
    fmt.Println("len:", len(m))  
 
    delete(m, "k2")
    fmt.Println("map:", m)  
 
    _, prs := m["k2"]
    fmt.Println("prs:", prs)  
 
    n := map[string]int{"foo": 1, "bar": 2}
    fmt.Println("map:", n)
}  
```
