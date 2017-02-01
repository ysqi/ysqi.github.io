
---
date: 2016-12-31T11:34:43+08:00
title: "介绍GDB调试Go"
description: ""
disqus_identifier: 1485833683332310663
slug: "jie-shao--GDB-diao-shi--Go"
source: "https://segmentfault.com/a/1190000000664422"
tags: 
- gdb 
- debug 
- golang 
topics:
- 编程语言与开发
---

> 注：本文作者是
> [YANN](http://lincolnloop.com/blog/by-author/yml/)，原文是
> [Introduction to Go Debugging with
> GDB](http://lincolnloop.com/blog/introduction-go-debugging-gdb/)

在过去的 4 年中，我花了我绝大部分的时间用来写，读以及调试 Python 或
JavaScript 代码。在学习 Go
的过程中，像穿着一双有小石子的鞋子在美丽的山中远行。很多事情给我留下了深刻的印象，但是使用
`println` 调试我的代码在过去走的太远了。在 Python
中，当代码在运行的时候，我们使用 `pdb/ipdb` 调试它，JavaScript
提供了类似的工具。在这些年中，这个模式已经变成了我工作流中非常重要的一部分了。

今天，我认识到 Go 已经内建支持 [Gnu debugger (aka
GDB)](http://sourceware.org/gdb/current/onlinedocs/gdb/)。

为了这篇文章，我们使用以下这个简单的程序：

    package main

    import (
        "fmt"
        "time"
    )

    func counting(c chan<- int) {
        for i := 0; i < 10; i++ {
            time.Sleep(2 * time.Second)
            c <- i
        }
        close(c)
    }

    func main() {
        msg := "Starting main"
        fmt.Println(msg)
        bus := make(chan int)
        msg = "starting a gofunc"
        go counting(bus)
        for count := range bus {
            fmt.Println("count:", count)
        }
    }

为了使用 GDB，你需要使用 `-gcflags ”-N -l”`
选项编译你的程序，这些选项阻止编译器使用内联函数和变量。

    go build -gcflags "-N -l" gdbsandbox.go

这是一个交互式的调试 GDB 的会话示例：

    yml@simba$  gdb gdbsandbox 
    GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2) 7.4-2012.04
    Copyright (C) 2012 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

首先，我们运行我们的程序：

    (gdb) run
    Starting program: /home/yml/Developments/go/src/gdbsandbox/gdbsandbox 
    Starting main
    count: 0
    count: 1
    count: 2
    [...]
    count: 9
    [Inferior 1 (process 13507) exited normally]

现在我们知道怎样运行我们的程序，我们可能想设置一个断点。

    (gdb) help break 
    Set breakpoint at specified line or function.
    break [LOCATION] [thread THREADNUM] [if CONDITION]
    LOCATION may be a line number, function name, or "*" and an address.
    [...]
    (gdb) break 22
    Breakpoint 1 at 0x400d7a: file /home/yml/Developments/go/src/gdbsandbox/gdbsandbox.go, line 22.
    (gdb) run
    Starting program: /home/yml/Developments/go/src/gdbsandbox/gdbsandbox 
    Starting main
    [New LWP 13672]
    [Switching to LWP 13672]
    Breakpoint 1, main.main () at /home/yml/Developments/go/src/gdbsandbox/gdbsandbox.go:22
    22              for count := range bus {
    (gdb) 

一旦 GDB 在你的断点处停止，你将看到上下文：

    (gdb) help list
    List specified function or line.
    With no argument, lists ten more lines after or around previous listing.
    "list -" lists the ten lines before a previous ten-line listing.
    [...]
    (gdb) list
    17              msg := "Starting main"
    18              fmt.Println(msg)
    19              bus := make(chan int)
    20              msg = "starting a gofunc"
    21              go counting(bus)
    22              for count := range bus {
    23                      fmt.Println("count:", count)
    24              }
    25      }

你也可以检查变量：

    (gdb) help print
    Print value of expression EXP.
    Variables accessible are those of the lexical environment of the selected
    stack frame, plus all those whose scope is global or an entire file.
    [...]
    (gdb) print msg
    $1 = "starting a gofunc"

在代码的早期，我们启动了一个 `goroutine`，当我执行到第 10
行的时候，我想查看我程序中的这部分。

    (gdb) break 10
    Breakpoint 3 at 0x400c28: file /home/yml/Developments/go/src/gdbsandbox/gdbsandbox.go, line 10.
    (gdb) help continue
    Continue program being debugged, after signal or breakpoint.
    If proceeding from breakpoint, a number N may be used as an argument,
    which means to set the ignore count of that breakpoint to N - 1 (so that
    the breakpoint won't break until the Nth time it is reached).
    [...]
    (gdb) continue
    Continuing.

在今天我们要做的最后一件事情就是在运行期改变一个变量的值：

    Breakpoint 3, main.counting (c=0xf840001a50) at /home/yml/Developments/go/src/gdbsandbox/gdbsandbox.go:10
    10                      time.Sleep(2 * time.Second)
    (gdb) help whatis
    Print data type of expression EXP.
    Only one level of typedefs is unrolled.  See also "ptype".
    (gdb) whatis count
    type = int
    (gdb) print count
    $3 = 1
    (gdb) set variable count=3
    (gdb) print count
    $4 = 3
    (gdb) c
    Continuing.
    count: 3

我们仅仅覆盖了以下命令：

-   list
-   next
-   print
-   continue
-   break
-   whatis
-   set variable =

这几乎只是涉及到了使用 GDB 的表面知识，如果你想学习关于 GDB
更多的知识，这里有更多的链接：

-   <http://sourceware.org/gdb/current/onlinedocs/gdb/>
-   <http://golang.org/doc/gdb>


