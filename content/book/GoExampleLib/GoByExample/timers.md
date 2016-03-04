---
book_chapter: "0132"
book_chapter_name: "定时器"
book_name: "Go示例大全"
date: "2016-03-04T13:13:25+08:00"
description: ""
disqus_identifier: "book000201032"
slug: "timers"
title: "Go示例大全-定时器"
codeurl: "https://wide.b3log.org/playground/48c8fdbc1f1f242831665516e534ff80.go"
---
 
我们常常需要在后面一个时刻运行 Go 代码，或者在某段时间间隔内重复运行。Go 的内置 _定时器_ 和 _打点器_ 特性让这些很容易实现。我们将先学习定时器，然后再学习[打点器](../tickers/)。







定时器表示在未来某一时刻的独立事件。你告诉定时器需要等待的时间，然后它将提供一个用于通知的通道。这里的定时器将等待 2 秒。

`<-timer1.C` 直到这个定时器的通道 `C` 明确的发送了定时器失效的值之前，将一直阻塞。

如果你需要的仅仅是单纯的等待，你需要使用 `time.Sleep`。定时器是有用原因之一就是你可以在定时器失效之前，取消这个定时器。这是一个例子
 

```go
package main  
import "time"
import "fmt"  
 func main() {  
 
    timer1 := time.NewTimer(time.Second * 2)  
 
    <-timer1.C
    fmt.Println("Timer 1 expired")  
 
    timer2 := time.NewTimer(time.Second)
    go func() {
        <-timer2.C
        fmt.Println("Timer 2 expired")
    }()
    stop2 := timer2.Stop()
    if stop2 {
        fmt.Println("Timer 2 stopped")
    }
}  
```
