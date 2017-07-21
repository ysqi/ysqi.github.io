
---
date: 2017-02-17T08:17:15+08:00
title: "Go1_8rc3源代码学习:token"
description: ""
disqus_identifier: 1487290635119540012
slug: "Go-1_8rc3-yuan-dai-ma-xue-xi-:token"
source: "https://segmentfault.com/a/1190000008312707"
tags: 
- 编译原理 
- 编译器 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

前言
====

token package 包含了 golang 词法分析相关的数据结构和方法，源代码位于
\<go-src\>/src/go/token

token.go
========

源代码中的注释很赞！

Token type
----------

Token is the set of lexical tokens of the Go programming language

    type Token int

tokens
------

The list of tokens（token ids）

    const (
        // Special tokens
        ILLEGAL Token = iota
        EOF
        COMMENT

        literal_begin
        ...
        literal_end

        operator_beg
        ...
        operator_end

        keyword_beg
        ...
        keyword_end
    )

使用 const 定义了 Go 语言 tokens，这里有一个地方值得学习：使用 xxx\_beg
和 xxx\_end 这一对伪 token 作为不同的 token group 分界，方便快速判断
token 类型，比如判断 token id 是否是一个关键字

    func (tok Token) IsKeyword() bool { return keyword_beg < tok && tok < keyword_end }

接下来是 token 对应的字符串描述（token string） 和 上述的 const 一一对应

    var tokens = [...]string {
        ILLEGAL: "ILLEGAL",

        EOF:     "EOF",
        COMMENT: "COMMENT",
        ...
    }

### 根据 token id 查询 token string

查询 tokens 数组，在此之前检查数组越界

    func (tok Token) String() string {
        s := ""
        if 0 <= tok && tok < Token(len(tokens)) {
            s = tokens[tok]
        }
        if s == "" {
            s = "token(" + strconv.Itoa(int(tok)) + ")"
        }
        return s
    }

keywords
--------

使用 map 保存关键字和 token id 对应关系

    var keywords map[string]Token

查询一个 string 是标识符还是关键字

    func Lookup(ident string) Token {
        if tok, is_keyword := keywords[ident]; is_keyword {
            return tok
        }
        return IDENT
    }

position.go
===========

Position type
-------------

Position describes an arbitrary source position including the file,
line, and column location

    type Position struct {
        Filename string // filename, if any
        Offset   int    // offset, starting at 0
        Line     int    // line number, starting at 1
        Column   int    // column number, starting at 1 (byte count)
    }

File type
---------

A File is a handle for a file belonging to a FileSet，it has a name,
size and line offset table

    type File struct {
        set  *FileSet
        name string // file name as provided to AddFile
        base int    // Pos value range for this file is [base...base+size]
        size int    // file size as provided to AddFile

        lines []int 
        infos []lineInfo
    }

-   lines and infos are protected by set.mutex

-   lines contains the offset of the first character for each line (the
    first entry is always 0)

### lines（line table）

有多种方法可以设置（初始化）File line table，比如根据文件内容

    func (f *File) SetLinesForContent(content []byte) {
        var lines []int
        line := 0
        for offset, b := range content {
            if line >= 0 {
                lines = append(lines, line)
            }
            line = -1
            if b == '\n' {
                line = offset + 1
            }
        }

        // set lines table
        f.set.mutex.Lock()
        f.lines = lines
        f.set.mutex.Unlock()
    }

-   lines 被定义成一个 slice，由于初始化时 line = 0，所以第一个被添加到
    lines 中的值为 0

-   当检测到一个换行符 'n' 时保存新行 offset + 1 到 line，这里 +1
    是为了跳过 n 字符

-   设置 File lines 字段时使用了 FileSet 的 mutex 进行锁保护

Pos type
--------

Pos is a compact encoding of a source position within a file set.

    type Pos int

这里的 Pos 容易和上文的 Position 混淆，Position 描述的是 File
内的位置信息，Pos 描述的是 FileSet
内的（编码后的）位置信息，它们之间可以相互转换

    func (f *File) PositionFor(p Pos, adjusted bool) (pos Position) {
        if p != NoPos {
            if int(p) < f.base || int(p) > f.base+f.size {
                panic("illegal Pos value")
            }
            pos = f.position(p, adjusted)
        }
        return
    }

    func (f *File) position(p Pos, adjusted bool) (pos Position) {
        offset := int(p) - f.base
        pos.Offset = offset
        pos.Filename, pos.Line, pos.Column = f.unpack(offset, adjusted)
        return
    }

如上文所述，File 的 base 属性代表 File 在 FileSet 中的"起始"位置，所以
position 方法中 int(p) - f.base 得到 p 所代表的 Position
在文件的偏移量，f.unpack 根据得到的偏移量对 lines 和 lineInfos
进行二分查找计算行，列偏移量

    func (f *File) unpack(offset int, adjusted bool) (filename string, line, column int) {
        filename = f.name
        if i := searchInts(f.lines, offset); i >= 0 {
            line, column = i+1, offset-f.lines[i]+1
        }
        if adjusted && len(f.infos) > 0 {
            // almost no files have extra line infos
            if i := searchLineInfos(f.infos, offset); i >= 0 {
                alt := &f.infos[i]
                filename = alt.Filename
                if i := searchInts(f.lines, alt.Offset); i >= 0 {
                    line += alt.Line - i - 1
                }
            }
        }
        return
    }

FileSet type
------------

总结
====

