---
book_chapter: "0137"
book_chapter_name: "互斥锁"
book_name: "Go示例大全"
date: "2016-03-04T13:13:27+08:00"
description: ""
disqus_identifier: "book000201037"
slug: "mutexes"
title: "Go示例大全-互斥锁"
codeurl: "https://wide.b3log.org/playground/ca2a201513bd25a98f03e4f8c668976b.go"
---
 
在前面的例子中，我们看到了如何使用原子操作来管理简单的计数器。对于更加复杂的情况，我们可以使用一个_[互斥锁](http://zh.wikipedia.org/wiki/%E4%BA%92%E6%96%A5%E9%94%81)_来在 Go 协程间安全的访问数据。







在我们的例子中，`state` 是一个 map。

这里的 `mutex` 将同步对 `state` 的访问。

we'll see later, `ops` will count how manyoperations we perform against the state.为了比较基于互斥锁的处理方式和我们后面将要看到的其他方式，`ops` 将记录我们对 state 的操作次数。

这里我们运行 100 个 Go 协程来重复读取 state。

每次循环读取，我们使用一个键来进行访问，`Lock()` 这个 `mutex` 来确保对 `state` 的独占访问，读取选定的键的值，`Unlock()` 这个mutex，并且 `ops` 值加 1。

为了确保这个 Go 协程不会再调度中饿死，我们在每次操作后明确的使用 `runtime.Gosched()`进行释放。这个释放一般是自动处理的，像例如每个通道操作后或者 `time.Sleep` 的阻塞调用后相似，但是在这个例子中我们需要手动的处理。

同样的，我们运行 10 个 Go 协程来模拟写入操作，使用和读取相同的模式。

让这 10 个 Go 协程对 `state` 和 `mutex` 的操作运行 1 s。

获取并输出最终的操作计数。

对 `state` 使用一个最终的锁，显示它是如何结束的。
 

```go
package main  
import (
    "fmt"
    "math/rand"
    "runtime"
    "sync"
    "sync/atomic"
    "time"
)  
 func main() {  
 
    var state = make(map[int]int)  
 
    var mutex = &sync.Mutex{}  
 
    var ops int64 = 0  
 
    for r := 0; r < 100; r++ {
        go func() {
            total := 0
            for {  
 
                key := rand.Intn(5)
                mutex.Lock()
                total += state[key]
                mutex.Unlock()
                atomic.AddInt64(&ops, 1)  
 
                runtime.Gosched()
            }
        }()
    }  
 
    for w := 0; w < 10; w++ {
        go func() {
            for {
                key := rand.Intn(5)
                val := rand.Intn(100)
                mutex.Lock()
                state[key] = val
                mutex.Unlock()
                atomic.AddInt64(&ops, 1)
                runtime.Gosched()
            }
        }()
    }  
 
    time.Sleep(time.Second)  
 
    opsFinal := atomic.LoadInt64(&ops)
    fmt.Println("ops:", opsFinal)  
 
    mutex.Lock()
    fmt.Println("state:", state)
    mutex.Unlock()
}  
```
