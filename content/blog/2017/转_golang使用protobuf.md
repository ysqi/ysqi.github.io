
---
date: 2017-05-24T09:17:29+08:00
title: "golang使用protobuf"
description: ""
disqus_identifier: 1495588649858245274
slug: "golangshi-yong-protobuf"
source: "https://segmentfault.com/a/1190000009277748"
tags: 
- protobuf 
- golang 
topics:
- 编程语言与开发
---

为什么要使用protobuf
====================

最近的项目中，一直使用Json做数据传输。Json用起来的确很方便。但相对于protobuf数据量更大些。做一个移动端应用，为用户省点流量还是很有必要的。正好也可以学习一下protobuf的使用

跟Json相比protobuf性能更高，更加规范

-   编解码速度快，数据体积小

-   使用统一的规范，不用再担心大小写不同导致解析失败等蛋疼的问题了

但也失去了一些便利性

-   改动协议字段，需要重新生成文件。

-   数据没有可读性

安装
====

在go中使用protobuf，有两个可选用的包goprotobuf（go官方出品）和gogoprotobuf。\
gogoprotobuf完全兼容google
protobuf，它生成的代码质量和编解码性能均比goprotobuf高一些

安装protoc
----------

首先去[https://github.com/google/pro...](https://github.com/google/protobuf/releases)
上下载protobuf的编译器protoc，windows上可以直接下到exe文件(linux则需要编译)，最后将下载好的可执行文件拷贝到\$GOPATH的bin目录下(\$GOPATH/bin目录最好添加到系统环境变量里)

安装protobuf库文件
------------------

    go get github.com/golang/protobuf/proto

goprotobuf
----------

### 安装插件

    go get github.com/golang/protobuf/protoc-gen-go

### 生成go文件

    protoc --go_out=. *.proto

gogoprotobuf
------------

### 安装插件

gogoprotobuf有两个插件可以使用

-   protoc-gen-gogo：和protoc-gen-go生成的文件差不多，性能也几乎一样(稍微快一点点)

-   protoc-gen-gofast：生成的文件更复杂，性能也更高(快5-7倍)

<!-- -->

    //gogo
    go get github.com/gogo/protobuf/protoc-gen-gogo

    //gofast
    go get github.com/gogo/protobuf/protoc-gen-gofast

### 安装gogoprotobuf库文件

    go get github.com/gogo/protobuf/proto
    go get github.com/gogo/protobuf/gogoproto  //这个不装也没关系

### 生成go文件

    //gogo
    protoc --gogo_out=. *.proto

    //gofast
    protoc --gofast_out=. *.proto

性能测试
========

这里只是简单的用go test测试了一下

    //goprotobuf
    "编码"：447ns/op
    "解码"：422ns/op

    //gogoprotobuf-go
    "编码"：433ns/op
    "解码"：427ns/op

    //gogoprotobuf-fast
    "编码"：112ns/op
    "解码"：112ns/op

go\_protobuf的简单使用
======================

test.proto
----------

    syntax = "proto3";  //指定版本，必须要写（proto3、proto2）  
    package proto;

    enum FOO 
    { 
        X = 0; 
    };

    //message是固定的。UserInfo是类名，可以随意指定，符合规范即可
    message UserInfo{
        string message = 1;   //消息
        int32 length = 2;    //消息大小
        int32 cnt = 3;      //消息计数
    }

client\_protobuf.go
-------------------

    package main

    import (
        "bufio"
        "fmt"
        "net"
        "os"
        stProto "proto"
        "time"

        //protobuf编解码库,下面两个库是相互兼容的，可以使用其中任意一个
        "github.com/golang/protobuf/proto"
        //"github.com/gogo/protobuf/proto"
    )

    func main() {
        strIP := "localhost:6600"
        var conn net.Conn
        var err error

        //连接服务器
        for conn, err = net.Dial("tcp", strIP); err != nil; conn, err = net.Dial("tcp", strIP) {
            fmt.Println("connect", strIP, "fail")
            time.Sleep(time.Second)
            fmt.Println("reconnect...")
        }
        fmt.Println("connect", strIP, "success")
        defer conn.Close()

        //发送消息
        cnt := 0
        sender := bufio.NewScanner(os.Stdin)
        for sender.Scan() {
            cnt++
            stSend := &stProto.UserInfo{
                Message: sender.Text(),
                Length:  *proto.Int(len(sender.Text())),
                Cnt:     *proto.Int(cnt),
            }

            //protobuf编码
            pData, err := proto.Marshal(stSend)
            if err != nil {
                panic(err)
            }

            //发送
            conn.Write(pData)
            if sender.Text() == "stop" {
                return
            }
        }
    }

server\_protobuf.go
-------------------

    package main

    import (
        "fmt"
        "net"
        "os"
        stProto "proto"

        //protobuf编解码库,下面两个库是相互兼容的，可以使用其中任意一个
        "github.com/golang/protobuf/proto"
        //"github.com/gogo/protobuf/proto"
    )

    func main() {
        //监听
        listener, err := net.Listen("tcp", "localhost:6600")
        if err != nil {
            panic(err)
        }

        for {
            conn, err := listener.Accept()
            if err != nil {
                panic(err)
            }
            fmt.Println("new connect", conn.RemoteAddr())
            go readMessage(conn)
        }
    }

    //接收消息
    func readMessage(conn net.Conn) {
        defer conn.Close()
        buf := make([]byte, 4096, 4096)
        for {
            //读消息
            cnt, err := conn.Read(buf)
            if err != nil {
                panic(err)
            }

            stReceive := &stProto.UserInfo{}
            pData := buf[:cnt]

            //protobuf解码
            err = proto.Unmarshal(pData, stReceive)
            if err != nil {
                panic(err)
            }

            fmt.Println("receive", conn.RemoteAddr(), stReceive)
            if stReceive.Message == "stop" {
                os.Exit(1)
            }
        }
    }

