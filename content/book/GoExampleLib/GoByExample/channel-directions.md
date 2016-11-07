---
book_chapter: "0126"
book_chapter_name: "通道方向"
book_name: "Go示例大全"
date: "2016-03-04T13:13:23+08:00"
description: ""
disqus_identifier: "book000201026"
slug: "channel-directions"
title: "Go示例大全-通道方向"
codeurl: "https://wide.b3log.org/playground/4e6c7f3a837ddd646433f08c67ebab90.go"
---
 
当使用通道作为函数的参数时，你可以指定这个通道是不是只用来发送或者接收值。这个特性提升了程序的类型安全性。





`ping` 函数定义了一个只允许发送数据的通道。尝试使用这个通道来接收数据将会得到一个编译时错误。

`pong` 函数允许通道（`pings`）来接收数据，另一通道（`pongs`）来发送数据。


 

```Go
package main  
import "fmt"  
 
func ping(pings chan<- string, msg string) {
    pings <- msg
}  
 
func pong(pings <-chan string, pongs chan<- string) {
    msg := <-pings
    pongs <- msg
}  
 func main() {
    pings := make(chan string, 1)
    pongs := make(chan string, 1)
    ping(pings, "passed message")
    pong(pings, pongs)
    fmt.Println(<-pongs)
}  
```
