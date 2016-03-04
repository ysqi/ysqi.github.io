---
book_chapter: "0142"
book_chapter_name: "Defer"
book_name: "Go示例大全"
date: "2016-03-04T13:13:29+08:00"
description: ""
disqus_identifier: "book000201042"
slug: "defer"
title: "Go示例大全-Defer"
codeurl: "https://wide.b3log.org/playground/369fe2a1b0067e46dd65cd6b15cc3ec2.go"
---
 
_Defer_ 被用来确保一个函数调用在程序执行结束前执行。同样用来执行一些清理工作。 `defer` 用在像其他语言中的`ensure` 和 `finally`用到的地方。





假设我们想要创建一个文件，向它进行写操作，然后在结束时关闭它。这里展示了如何通过 `defer` 来做到这一切。

在 `closeFile` 后得到一个文件对象，我们使用 defer通过 `closeFile` 来关闭这个文件。这会在封闭函数（`main`）结束时执行，就是 `writeFile` 结束后。








 

```go
package main  
import "fmt"
import "os"  
 
func main() {  
 
    f := createFile("/tmp/defer.txt")
    defer closeFile(f)
    writeFile(f)
}  
 func createFile(p string) *os.File {
    fmt.Println("creating")
    f, err := os.Create(p)
    if err != nil {
        panic(err)
    }
    return f
}  
 func writeFile(f *os.File) {
    fmt.Println("writing")
    fmt.Fprintln(f, "data")  
 }  
 func closeFile(f *os.File) {
    fmt.Println("closing")
    f.Close()
}  
```
