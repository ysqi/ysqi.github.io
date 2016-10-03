---
book_chapter: "0135"
book_chapter_name: "速率限制"
book_name: "Go示例大全"
date: "2016-03-04T13:13:27+08:00"
description: ""
disqus_identifier: "book000201035"
slug: "rate-limiting"
title: "Go示例大全-速率限制"
codeurl: "https://wide.b3log.org/playground/6d6430e91c84550cd484de47b6b59c96.go"
---
 
[_速率限制(英)_](http://en.wikipedia.org/wiki/Rate_limiting) 是一个重要的控制服务资源利用和质量的途径。Go 通过 Go 协程、通道和[打点器](../tickers/)优美的支持了速率限制。







首先我们将看一下基本的速率限制。假设我们想限制我们接收请求的处理，我们将这些请求发送给一个相同的通道。

这个 `limiter` 通道将每 200ms 接收一个值。这个是速率限制任务中的管理器。

通过在每次请求前阻塞 `limiter` 通道的一个接收，我们限制自己每 200ms 执行一次请求。

有时候我们想临时进行速率限制，并且不影响整体的速率控制我们可以通过[通道缓冲](channel-buffering.html)来实现。这个 `burstyLimiter` 通道用来进行 3 次临时的脉冲型速率限制。

想将通道填充需要临时改变次的值，做好准备。

每 200 ms 我们将添加一个新的值到 `burstyLimiter`中，直到达到 3 个的限制。

现在模拟超过 5 个的接入请求。它们中刚开始的 3 个将由于受 `burstyLimiter` 的“脉冲”影响。
 

```go
package main  
import "time"
import "fmt"  
 func main() {  
 
    requests := make(chan int, 5)
    for i := 1; i <= 5; i++ {
        requests <- i
    }
    close(requests)  
 
    limiter := time.Tick(time.Millisecond * 200)  
 
    for req := range requests {
        <-limiter
        fmt.Println("request", req, time.Now())
    }  
 
    burstyLimiter := make(chan time.Time, 3)  
 
    for i := 0; i < 3; i++ {
        burstyLimiter <- time.Now()
    }  
 
    go func() {
        for t := range time.Tick(time.Millisecond * 200) {
            burstyLimiter <- t
        }
    }()  
 
    burstyRequests := make(chan int, 5)
    for i := 1; i <= 5; i++ {
        burstyRequests <- i
    }
    close(burstyRequests)
    for req := range burstyRequests {
        <-burstyLimiter
        fmt.Println("request", req, time.Now())
    }
}  
```
