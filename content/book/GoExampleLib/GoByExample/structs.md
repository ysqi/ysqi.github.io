---
book_chapter: "0118"
book_chapter_name: "结构体"
book_name: "Go示例大全"
date: "2016-03-04T13:13:20+08:00"
description: ""
disqus_identifier: "book000201018"
slug: "structs"
title: "Go示例大全-结构体"
codeurl: "https://wide.b3log.org/playground/6a079d8820802f56a155a71257304359.go"
---
 
Go 的_结构体_ 是各个字段字段的类型的集合。这在组织数据时非常有用。





这里的 `person` 结构体包含了 `name` 和 `age` 两个字段。



使用这个语法创建了一个新的结构体元素。

你可以在初始化一个结构体元素时指定字段名字。

省略的字段将被初始化为零值。

`&` 前缀生成一个结构体指针。

使用点来访问结构体字段。

也可以对结构体指针使用`.` - 指针会被自动解引用。

结构体是可变的。
 

```Go
package main  
import "fmt"  
 
type person struct {
    name string
    age  int
}  
 func main() {  
 
    fmt.Println(person{"Bob", 20})  
 
    fmt.Println(person{name: "Alice", age: 30})  
 
    fmt.Println(person{name: "Fred"})  
 
    fmt.Println(&person{name: "Ann", age: 40})  
 
    s := person{name: "Sean", age: 50}
    fmt.Println(s.name)  
 
    sp := &s
    fmt.Println(sp.age)  
 
    sp.age = 51
    fmt.Println(sp.age)
}  
```
