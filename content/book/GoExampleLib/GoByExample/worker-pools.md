---
book_chapter: "0134"
book_chapter_name: "工作池"
book_name: "Go示例大全"
date: "2016-03-04T13:13:26+08:00"
description: ""
disqus_identifier: "book000201034"
slug: "worker-pools"
title: "Go示例大全-工作池"
codeurl: "https://wide.b3log.org/playground/7ea34ba4552e57c2c6db572c497a2598.go"
---
 
在这个例子中，我们将看到如何使用 Go  协程和通道实现一个_工作池_ 。





这是我们将要在多个并发实例中支持的任务了。这些执行者将从 `jobs` 通道接收任务，并且通过 `results` 发送对应的结果。我们将让每个任务间隔 1s 来模仿一个耗时的任务。



为了使用 worker 工作池并且收集他们的结果，我们需要2 个通道。

这里启动了 3 个 worker，初始是阻塞的，因为还没有传递任务。

这里我们发送 9 个 `jobs`，然后 `close` 这些通道来表示这些就是所有的任务了。



最后，我们收集所有这些人物的返回值。
 

```Go
package main  
import "fmt"
import "time"  
 
func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker", id, "processing job", j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}  
 func main() {  
 
    jobs := make(chan int, 100)
    results := make(chan int, 100)  
 
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }  
 
     for j := 1; j <= 9; j++ {
        jobs <- j
    }
    close(jobs)  
 
    for a := 1; a <= 9; a++ {
        <-results
    }
}  
```
