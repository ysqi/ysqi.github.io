
---
date: 2017-02-17T08:17:14+08:00
title: "Go1_8rc3源代码学习:scanner"
description: ""
disqus_identifier: 1487290634642834341
slug: "Go-1_8rc3-yuan-dai-ma-xue-xi-:scanner"
source: "https://segmentfault.com/a/1190000008314192"
tags: 
- golang 
- 编译原理 
- 编译器 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

前言
====

scanner package 包含了 golang 词法分析器相关的数据结构和方法，源代码位于
\<go-src\>/src/go/scanner

example\_test.go
----------------

example\_test.go 包含了一个使用 scanner 包的示例方法，该方法对 Euler
公式进行词法扫描

    func ExampleScanner_Scan() {
        // src is the input that we want to tokenize.
        src := []byte("cos(x) + 1i*sin(x) // Euler")

        // Initialize the scanner.
        var s scanner.Scanner
        fset := token.NewFileSet()                      // positions are relative to fset
        file := fset.AddFile("", fset.Base(), len(src)) // register input "file"
        s.Init(file, src, nil /* no error handler */, scanner.ScanComments)

        // Repeated calls to Scan yield the token sequence found in the input.
        for {
            pos, tok, lit := s.Scan()
            if tok == token.EOF {
                break
            }
            fmt.Printf("%s\t%s\t%q\n", fset.Position(pos), tok, lit)
        }
    }

注释已经说明的很清楚了

-   创建 FileSet

-   添加 File

-   初始化 Scanner 使用 File，src（待解析的字符串）

-   遍历所有的 token，使用 Scanner.Scan

总结
====

