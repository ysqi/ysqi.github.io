---
book_chapter: "0133"
book_chapter_name: "打点器"
book_name: "Go示例大全"
date: "2016-03-04T13:13:26+08:00"
description: ""
disqus_identifier: "book000201033"
slug: "tickers"
title: "Go示例大全-打点器"
codeurl: "https://wide.b3log.org/playground/256e34198d173429b7a15e01b396ebf8.go"
---
 
[定时器](../timers/) 是当你想要在未来某一刻执行一次时使用的 - _打点器_ 则是当你想要在固定的时间间隔重复执行准备的。这里是一个打点器的例子，它将定时的执行，直到我们将它停止。







打点器和定时器的机制有点相似：一个通道用来发送数据。这里我们在这个通道上使用内置的 `range` 来迭代值每隔500ms 发送一次的值。

打点器可以和定时器一样被停止。一旦一个打点停止了，将不能再从它的通道中接收到值。我们将在运行后 1600ms停止这个打点器。
 

```Go
package main  
import "time"
import "fmt"  
 func main() {  
 
    ticker := time.NewTicker(time.Millisecond * 500)
    go func() {
        for t := range ticker.C {
            fmt.Println("Tick at", t)
        }
    }()  
 
    time.Sleep(time.Millisecond * 1600)
    ticker.Stop()
    fmt.Println("Ticker stopped")
}  
```
