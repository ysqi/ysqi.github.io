---
book_chapter: "0163"
book_chapter_name: "执行进程"
book_name: "Go示例大全"
date: "2016-03-04T13:13:37+08:00"
description: ""
disqus_identifier: "book000201063"
slug: "execing-processes"
title: "Go示例大全-执行进程"
codeurl: "https://wide.b3log.org/playground/ad046c8e2a5082ef897d9ad63518556f.go"
---
 
在前面的例子中，我们了解了[生成外部进程](../spawning-processes/)的知识，当我们需要访问外部进程时时需要这样做，但是有时候，我们只想用其他的（也许是非 Go 程序）来完全替代当前的 Go 进程。这时候，我们可以使用经典的 <a href="http://en.wikipedia.org/wiki/Exec_(operating_system)"><code>exec</code></a>方法的 Go 实现。







在我们的例子中，我们将执行 `ls` 命令。Go 需要提供我们需要执行的可执行文件的绝对路径，所以我们将使用`exec.LookPath` 来得到它（大概是 `/bin/ls`）。

`Exec` 需要的参数是切片的形式的（不是放在一起的一个大字符串）。我们给 `ls` 一些基本的参数。注意，第一个参数需要是程序名。

`Exec` 同样需要使用[环境变量](environment-variables.html)。这里我们仅提供当前的环境变量。

这里是 `os.Exec` 调用。如果这个调用成功，那么我们的进程将在这里被替换成 `/bin/ls -a -l -h` 进程。如果存在错误，那么我们将会得到一个返回值。
 

```Go
package main  
import "syscall"
import "os"
import "os/exec"  
 func main() {  
 
    binary, lookErr := exec.LookPath("ls")
    if lookErr != nil {
        panic(lookErr)
    }  
 
    args := []string{"ls", "-a", "-l", "-h"}  
 
    env := os.Environ()  
 
    execErr := syscall.Exec(binary, args, env)
    if execErr != nil {
        panic(execErr)
    }
}  
```
