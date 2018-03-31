
---
date: 2016-12-31T11:32:54+08:00
title: "golang调用cgocoredump获得方法"
description: ""
disqus_identifier: 1485833574858323582
slug: "golang-diao-yong--cgo-coredump-huo-de-fang-fa"
source: "https://segmentfault.com/a/1190000007388342"
tags: 
- golang 
- cgo 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

写一个错误的c程序
=================

    package dlsym

    import "testing"

    func Test_intercept(t *testing.T) {
        Intercept("gethostbyname\x00")
    }

    package dlsym

    // #cgo CFLAGS: -I.
    // #include <stddef.h>
    // #include "dlsym_wrapper.h"
    import "C"
    import "unsafe"

    func Intercept(symbol string) {
        ptr := unsafe.Pointer(&([]byte(symbol)[0]))
        C.intercept((*C.char)(ptr), C.size_t(len(symbol)))
    }

    #include <dlfcn.h>
    #include <stddef.h>
    #include <stdio.h>

    void intercept(char *symbol, size_t symbol_len) {
        symbol = NULL; // will cause SIGSEGV
        printf("%s\n", symbol);
        fflush(stdout);
    }

编译测试为可执行文件
====================

    go test -c github.com/taowen/go-lib c/dlsym
    # will produce executable dlsym.test

这个是用于分析coredump的时候获得符号表使用的。

执行测试，获得coredump
======================

    GOTRACEBACK=crash ./dlsym.test
    # produced /tmp/core_dlsym.test.29937

如果找不到coredump的位置，执行之前先设置好coredump的写出条件

    echo '/tmp/core_%e.%p' | sudo tee /proc/sys/kernel/core_pattern
    ulimit -c unlimited # coredump can be any large

用gdb分析coredump
=================

    gdb dlsym.test /tmp/core_dlsym.test /tmp/core_dlsym.test.29937

-   用 `bt full` 查看所有的frame

-   用 `frame <number>` 查看指定的frame

-   用 `print <symbol>` 查看指定的变量的值



