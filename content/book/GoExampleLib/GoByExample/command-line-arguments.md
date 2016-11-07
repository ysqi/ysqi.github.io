---
book_chapter: "0159"
book_chapter_name: "命令行参数"
book_name: "Go示例大全"
date: "2016-03-04T13:13:36+08:00"
description: ""
disqus_identifier: "book000201059"
slug: "command-line-arguments"
title: "Go示例大全-命令行参数"
codeurl: "https://wide.b3log.org/playground/ac772340178a375b2c6f548625233a1f.go"
---
 
[_命令行参数_](http://en.wikipedia.org/wiki/Command-line_interface#Arguments)是指定程序运行参数的一个常见方式。例如，`go run hello.go`，程序 `go` 使用了 `run` 和 `hello.go` 两个参数。







`os.Args` 提供原始命令行参数访问功能。注意，切片中的第一个参数是该程序的路径，并且 `os.Args[1:]`保存所有程序的的参数。

你可以使用标准的索引位置方式取得单个参数的值。


 

```Go
package main  
import "os"
import "fmt"  
 func main() {  
 
    argsWithProg := os.Args
    argsWithoutProg := os.Args[1:]  
 
    arg := os.Args[3]  
     fmt.Println(argsWithProg)
    fmt.Println(argsWithoutProg)
    fmt.Println(arg)
}  
```
