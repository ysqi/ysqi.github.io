---
book_chapter: "0122"
book_chapter_name: "协程"
book_name: "Go示例大全"
date: "2016-03-04T13:13:22+08:00"
description: ""
disqus_identifier: "book000201022"
slug: "goroutines"
title: "Go示例大全-协程"
codeurl: "https://wide.b3log.org/playground/c48f8df45ad74e96aabc41a52c64efe0.go"
---
 
_Go 协程_ 在执行上来说是轻量级的线程。









假设我们有一个函数叫做 `f(s)`。我们使用一般的方式调并同时运行。

使用 `go f(s)` 在一个 Go 协程中调用这个函数。这个新的 Go 协程将会并行的执行这个函数调用。

你也可以为匿名函数启动一个 Go 协程。

现在这两个 Go 协程在独立的 Go 协程中异步的运行，所以我们需要等它们执行结束。这里的 `Scanln` 代码需要我们在程序退出前按下任意键结束。
 

```Go
package main  
import "fmt"  
 func f(from string) {
    for i := 0; i < 3; i++ {
        fmt.Println(from, ":", i)
    }
}  
 func main() {  
 
    f("direct")  
 
    go f("goroutine")  
 
    go func(msg string) {
        fmt.Println(msg)
    }("going")  
 
    var input string
    fmt.Scanln(&input)
    fmt.Println("done")
}  
```
