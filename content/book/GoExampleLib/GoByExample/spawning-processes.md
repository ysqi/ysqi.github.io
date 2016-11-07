---
book_chapter: "0162"
book_chapter_name: "生成进程"
book_name: "Go示例大全"
date: "2016-03-04T13:13:37+08:00"
description: ""
disqus_identifier: "book000201062"
slug: "spawning-processes"
title: "Go示例大全-生成进程"
codeurl: "https://wide.b3log.org/playground/1ed63b5355a932644ab1fae9ede67d40.go"
---
 
有时，我们的 Go 程序需要生成其他的，非 Go 进程。例如，这个网站的语法高亮是通过在 Go 程序中生成一个 [`pygmentize`](http://pygments.org/)来[实现的](https://github.com/everyx/gobyexample/blob/master/tools/generate.go)。让我们看一些关于 Go 生成进程的例子。







我们将从一个简单的命令开始，没有参数或者输入，仅打印一些信息到标准输出流。`exec.Command` 函数帮助我们创建一个表示这个外部进程的对象。

`.Output` 是另一个帮助我们处理运行一个命令的常见情况的函数，它等待命令运行完成，并收集命令的输出。如果没有出错，`dateOut` 将获取到日期信息的字节。

下面我们将看看一个稍复杂的例子，我们将从外部进程的`stdin` 输入数据并从 `stdout` 收集结果。

这里我们明确的获取输入/输出管道，运行这个进程，写入一些输入信息，读取输出的结果，最后等待程序运行结束。

上面的例子中，我们忽略了错误检测，但是你可以使用`if err != nil` 的方式来进行错误检查，我们也只收集`StdoutPipe` 的结果，但是你可以使用相同的方法收集`StderrPipe` 的结果。

注意，当我们需要提供一个明确的命令和参数数组来生成命令，和能够只需要提供一行命令行字符串相比，你想使用通过一个字符串生成一个完整的命令，那么你可以使用 `bash`命令的 `-c` 选项：
 

```Go
package main  
import "fmt"
import "io/ioutil"
import "os/exec"  
 func main() {  
 
    dateCmd := exec.Command("date")  
 
    dateOut, err := dateCmd.Output()
    if err != nil {
        panic(err)
    }
    fmt.Println("> date")
    fmt.Println(string(dateOut))  
 
    grepCmd := exec.Command("grep", "hello")  
 
    grepIn, _ := grepCmd.StdinPipe()
    grepOut, _ := grepCmd.StdoutPipe()
    grepCmd.Start()
    grepIn.Write([]byte("hello grep\ngoodbye grep"))
    grepIn.Close()
    grepBytes, _ := ioutil.ReadAll(grepOut)
    grepCmd.Wait()  
 
    fmt.Println("> grep hello")
    fmt.Println(string(grepBytes))  
 
    lsCmd := exec.Command("bash", "-c", "ls -a -l -h")
    lsOut, err := lsCmd.Output()
    if err != nil {
        panic(err)
    }
    fmt.Println("> ls -a -l -h")
    fmt.Println(string(lsOut))
}  
```
