
---
date: 2017-02-17T08:17:13+08:00
title: "Go1_8rc3源代码学习:parser"
description: ""
disqus_identifier: 1487290633944844674
slug: "Go-1_8rc3-yuan-dai-ma-xue-xi-:parser"
source: "https://segmentfault.com/a/1190000008317355"
tags: 
- golang 
- 编译原理 
- 语法分析 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

前言
====

parser package 包含了 golang 语法分析相关的数据结构和方法，源代码位于
\<go-src\>/src/go/parser

之前大概看了点 PHP 和 Ruby 的源代码，感叹 go 确实如宣传的一样，简洁如
C，parser.go 代码总共 几千行（Ruby 语法规则定义文件有 1w
多行），使用递归下降语法分析方法（感觉 go 语言的语法规则很适合递归下降）

example\_test.go
================

parser package 里面也有一个示例 example\_test.go，如何使用 parser

    func ExampleParseFile() {
        fset := token.NewFileSet() // positions are relative to fset

        // Parse the file containing this very example
        // but stop after processing the imports.
        f, err := parser.ParseFile(fset, "example_test.go", nil, parser.ImportsOnly)
        if err != nil {
            fmt.Println(err)
            return
        }

        // Print the imports from the file's AST.
        for _, s := range f.Imports {
            fmt.Println(s.Path.Value)
        }

        // output:
        //
        // "fmt"
        // "go/parser"
        // "go/token"
    }

parser struct
=============

The parser structure holds the parser's internal state.

    type parser struct {
        // 词法扫描相关字段
        file *token.File
        errors scanner.ErrorList
        scanner scanner.Scanner
        ...
        pos token.Pos   // token position
        tok token.Token // one token look-ahead
        lit string      // token literal
        ...
        // 作用域相关字段
        pkgScope *ast.Scope          // pkgScope.Outer == nil
        topScope *ast.Scope          // top-most scope; may be pkgScope
        unresolved []*ast.Ident      // unresolved identifiers
        imports    []*ast.ImportSpec // list of imports
        ...
    }

parser
结构体以小写字母开头，意味着它是一个仅供内部使用的数据结构，里面字段比较多，一时不明白用途关系不大，有个大概印象

总结
====

