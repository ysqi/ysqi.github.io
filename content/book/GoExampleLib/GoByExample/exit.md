---
book_chapter: "0165"
book_chapter_name: "退出"
book_name: "Go示例大全"
date: "2016-03-04T13:13:38+08:00"
description: ""
disqus_identifier: "book000201065"
slug: "exit"
title: "Go示例大全-退出"
codeurl: "https://wide.b3log.org/playground/6ac50269a9981f4616a38d92f28723bb.go"
---
 
使用 `os.Exit` 来立即进行带给定状态的退出。







当使用 `os.Exit` 时 `defer` 将_不会_ 执行，所以这里的 `fmt.Println`将永远不会被调用。

退出并且退出状态为 3。

注意，不像例如 C 语言，Go 不使用在 `main` 中返回一个整数来指明退出状态。如果你想以非零状态退出，那么你就要使用 `os.Exit`。
 

```Go
package main  
import "fmt"
import "os"  
 func main() {  
 
    defer fmt.Println("!")  
 
    os.Exit(3)
}  
 
```
