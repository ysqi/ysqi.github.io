
---
date: 2016-12-31T11:34:25+08:00
title: "Golang中JSON的使用"
description: ""
disqus_identifier: 1485833665178332554
slug: "Golangzhong-JSONde-shi-yong"
source: "https://segmentfault.com/a/1190000003034837"
tags: 
- golang 
- json 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

GO Json
=======

> **author: qcliu**\
> **date: 2015/07/21**

### Abstrct

介绍go语言中json的使用

### json

json是一种传输格式，类似与XML，与XML相比可读性略差，但是传输效率高。

### GO Json

go语言中提供了json的encoder，可以将数据结构转换为json格式。在使用之前，需要导入包

    import "encoding/json"

#### Encode

使用

    func NewEncoder(w io.Writer) *Encoder

创建一个json的encode。

    file, _ := os.Create("json.txt")
    enc := json.NewEncoder(file)
    err := enc.Encode(&v)

数据结构v会以json格式写入`json.txt`文件。

#### Decode

使用

    func NewDecoder(r io.Reader) *Decoder

创建一个json的decode。

    fp, _ os.Open("json.txt")
    dec := json.NewDecoder(fp)
    for {
        var V v
        err := dec.Decode(&v)
        if err != nil {
            break
        }
        //use v
    }

v是一个数据结构空间，decoder会将文件中的json格式按照v的定义转化，存在v中。

#### Example

    type Person struct {
        name string
        age int
    }

    type Student struct {
        p *Person
        sno int
    }

对于Student类型，虽然里面有一个指针，gojson一样可以处理。在encode与decode时，会自动的递归下降的进行格式转换。

### Summary

-   encoder与decoder像是在writer外面封装了一层。会根据指定的数据结构的格式进行读写。如果文件中的json格式与指定的数据结构的格式不一致会出现error。
-   在decoder的过程中，用一个for｛｝不停的读文件，直到出现error，代表文件结束。在for｛｝中，每次都要申请一个新的空间，存放从文件中读取出来的数据。


