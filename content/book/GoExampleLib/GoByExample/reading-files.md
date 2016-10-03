---
book_chapter: "0156"
book_chapter_name: "读文件"
book_name: "Go示例大全"
date: "2016-03-04T13:13:35+08:00"
description: ""
disqus_identifier: "book000201056"
slug: "reading-files"
title: "Go示例大全-读文件"
codeurl: "https://wide.b3log.org/playground/cd1964268d42a730e57c69f2e435a185.go"
---
 
读写文件在很多程序中都是必须的基本任务。首先我们看看一些读文件的例子。





读取文件需要经常进行错误检查，这个帮助方法可以精简下面的错误检查过程。



也许大部分基本的文件读取任务是将文件内容读取到内存中。

你经常会想对于一个文件是怎么读并且读取到哪一部分进行更多的控制。对于这个任务，从使用 `os.Open`打开一个文件获取一个 `os.File` 值开始。

从文件开始位置读取一些字节。这里最多读取 5 个字节，并且这也是我们实际读取的字节数。

你也可以 `Seek` 到一个文件中已知的位置并从这个位置开始进行读取。

`io` 包提供了一些可以帮助我们进行文件读取的函数。例如，上面的读取可以使用 `ReadAtLeast` 得到一个更健壮的实现。

没有内置的回转支持，但是使用 `Seek(0, 0)` 实现。

`bufio` 包实现了带缓冲的读取，这不仅对有很多小的读取操作的能提升性能，也提供了很多附加的读取函数。

任务结束后要关闭这个文件（通常这个操作应该在 `Open`操作后立即使用 `defer` 来完成）。


 

```go
package main  
import (
    "bufio"
    "fmt"
    "io"
    "io/ioutil"
    "os"
)  
 
func check(e error) {
    if e != nil {
        panic(e)
    }
}  
 func main() {  
 
    dat, err := ioutil.ReadFile("/tmp/dat")
    check(err)
    fmt.Print(string(dat))  
 
    f, err := os.Open("/tmp/dat")
    check(err)  
 
    b1 := make([]byte, 5)
    n1, err := f.Read(b1)
    check(err)
    fmt.Printf("%d bytes: %s\n", n1, string(b1))  
 
    o2, err := f.Seek(6, 0)
    check(err)
    b2 := make([]byte, 2)
    n2, err := f.Read(b2)
    check(err)
    fmt.Printf("%d bytes @ %d: %s\n", n2, o2, string(b2))  
 
    o3, err := f.Seek(6, 0)
    check(err)
    b3 := make([]byte, 2)
    n3, err := io.ReadAtLeast(f, b3, 2)
    check(err)
    fmt.Printf("%d bytes @ %d: %s\n", n3, o3, string(b3))  
 
    _, err = f.Seek(0, 0)
    check(err)  
 
    r4 := bufio.NewReader(f)
    b4, err := r4.Peek(5)
    check(err)
    fmt.Printf("5 bytes: %s\n", string(b4))  
 
    f.Close()  
 }  
```
