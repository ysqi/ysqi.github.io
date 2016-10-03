---
book_chapter: "0125"
book_chapter_name: "通道同步"
book_name: "Go示例大全"
date: "2016-03-04T13:13:23+08:00"
description: ""
disqus_identifier: "book000201025"
slug: "channel-synchronization"
title: "Go示例大全-通道同步"
codeurl: "https://wide.b3log.org/playground/5ccc5beb24084b1861e66b70e96f888a.go"
---
 
我们可以使用通道来同步 Go 协程间的执行状态。这里是一个使用阻塞的接受方式来等待一个 Go 协程的运行结束。





这是一个我们将要在 Go 协程中运行的函数。`done` 通道将被用于通知其他 Go 协程这个函数已经工作完毕。

发送一个值来通知我们已经完工啦。



运行一个 worker Go协程，并给予用于通知的通道。

程序将在接收到通道中 worker 发出的通知前一直阻塞。
 

```go
package main  
import "fmt"
import "time"  
 
func worker(done chan bool) {
    fmt.Print("working...")
    time.Sleep(time.Second)
    fmt.Println("done")  
 
    done <- true
}  
 func main() {  
 
    done := make(chan bool, 1)
    go worker(done)  
 
    <-done
}  
```
