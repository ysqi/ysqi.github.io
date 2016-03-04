---
book_chapter: "0127"
book_chapter_name: "通道选择器"
book_name: "Go示例大全"
date: "2016-03-04T13:13:24+08:00"
description: ""
disqus_identifier: "book000201027"
slug: "select"
title: "Go示例大全-通道选择器"
codeurl: "https://wide.b3log.org/playground/110ec463f8f4ab003413b3786ad054ce.go"
---
 
Go 的_通道选择器_ 让你可以同时等待多个通道操作。Go 协程和通道以及选择器的结合是 Go 的一个强大特性。







在我们的例子中，我们将从两个通道中选择。

各个通道将在若干时间后接收一个值，这个用来模拟例如并行的 Go 协程中阻塞的 RPC 操作

我们使用 `select` 关键字来同时等待这两个值，并打印各自接收到的值。
 

```go
package main  
import "time"
import "fmt"  
 func main() {  
 
    c1 := make(chan string)
    c2 := make(chan string)  
 
    go func() {
        time.Sleep(time.Second * 1)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(time.Second * 2)
        c2 <- "two"
    }()  
 
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
}  
```
