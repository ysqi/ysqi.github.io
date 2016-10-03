---
book_chapter: "0124"
book_chapter_name: "通道缓冲"
book_name: "Go示例大全"
date: "2016-03-04T13:13:22+08:00"
description: ""
disqus_identifier: "book000201024"
slug: "channel-buffering"
title: "Go示例大全-通道缓冲"
codeurl: "https://wide.b3log.org/playground/8ecfa0a5429391139e084ec927e57d6b.go"
---
 
默认通道是 _无缓冲_ 的，这意味着只有在对应的接收（`<- chan`）通道准备好接收时，才允许进行发送（`chan <-`）。_可缓存通道_允许在没有对应接收方的情况下，缓存限定数量的值。







这里我们 `make` 了一个通道，最多允许缓存 2 个值。

因为这个通道是有缓冲区的，即使没有一个对应的并发接收方，我们仍然可以发送这些值。

然后我们可以像前面一样接收这两个值。
 

```go
package main  
import "fmt"  
 func main() {  
 
    messages := make(chan string, 2)  
 
    messages <- "buffered"
    messages <- "channel"  
 
    fmt.Println(<-messages)
    fmt.Println(<-messages)
}  
```
