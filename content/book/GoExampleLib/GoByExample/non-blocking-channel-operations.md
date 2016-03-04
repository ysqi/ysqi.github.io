---
book_chapter: "0129"
book_chapter_name: "非阻塞通道操作"
book_name: "Go示例大全"
date: "2016-03-04T13:13:24+08:00"
description: ""
disqus_identifier: "book000201029"
slug: "non-blocking-channel-operations"
title: "Go示例大全-非阻塞通道操作"
codeurl: "https://wide.b3log.org/playground/d99e0e39ee9c478bc5b919d5dc9e1507.go"
---
 
常规的通过通道发送和接收数据是阻塞的。然而，我们可以使用带一个 `default` 子句的 `select` 来实现_非阻塞_ 的发送、接收，甚至是非阻塞的多路 `select`。







这里是一个非阻塞接收的例子。如果在 `messages` 中存在，然后 `select` 将这个值带入 `<-messages` `case`中。如果不是，就直接到 `default` 分支中。

一个非阻塞发送的实现方法和上面一样。

我们可以在 `default` 前使用多个 `case` 子句来实现一个多路的非阻塞的选择器。这里我们视图在 `messages`和 `signals` 上同时使用非阻塞的接受操作。
 

```go
package main  
import "fmt"  
 func main() {
    messages := make(chan string)
    signals := make(chan bool)  
 
    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    default:
        fmt.Println("no message received")
    }  
 
    msg := "hi"
    select {
    case messages <- msg:
        fmt.Println("sent message", msg)
    default:
        fmt.Println("no message sent")
    }  
 
    select {
    case msg := <-messages:
        fmt.Println("received message", msg)
    case sig := <-signals:
        fmt.Println("received signal", sig)
    default:
        fmt.Println("no activity")
    }
}  
```
