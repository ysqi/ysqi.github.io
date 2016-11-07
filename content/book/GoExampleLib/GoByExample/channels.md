---
book_chapter: "0123"
book_chapter_name: "通道"
book_name: "Go示例大全"
date: "2016-03-04T13:13:22+08:00"
description: ""
disqus_identifier: "book000201023"
slug: "channels"
title: "Go示例大全-通道"
codeurl: "https://wide.b3log.org/playground/e1e87c23a2b618cd25c88df8f8b30d8d.go"
---
 
_通道_ 是连接多个 Go 协程的管道。你可以从一个 Go 协程将值发送到通道，然后在别的 Go 协程中接收。







使用 `make(chan val-type)` 创建一个新的通道。通道类型就是他们需要传递值的类型。

使用 `channel <-` 语法 _发送_ 一个新的值到通道中。这里我们在一个新的 Go 协程中发送 `"ping"` 到上面创建的`messages` 通道中。

使用 `<-channel` 语法从通道中 _接收_ 一个值。这里将接收我们在上面发送的 `"ping"` 消息并打印出来。
 

```Go
package main  
import "fmt"  
 func main() {  
 
    messages := make(chan string)  
 
    go func() { messages <- "ping" }()  
 
    msg := <-messages
    fmt.Println(msg)
}  
```
