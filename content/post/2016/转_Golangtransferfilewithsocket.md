
---
date: 2016-12-31T11:33:24+08:00
title: "Golangtransferfilewithsocket"
description: ""
disqus_identifier: 1485833604580069658
slug: "Golang-transfer-file-with-socket"
source: "https://segmentfault.com/a/1190000005845010"
tags: 
- tcp 
- net 
- io 
- socket 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main
    import (
        "bufio"
        "code.google.com/p/mahonia"
        "fmt"
        "io"
        "net"
        "os"
    )
    func main() {
        fmt.Println("create a server or client?")
        reader := bufio.NewReader(os.Stdin)
        input, _, _ := reader.ReadLine()
        if string(input) == "server" {
            Server()
        }
        if string(input) == "client" {
            Client()
        } else {
            fmt.Println(Show("err arguments,entering again!.\r\n  alternaltive argument is server or client"))
            os.Exit(0)
        }
    }
    func Show(s string) string {
        enc := mahonia.NewEncoder("gbk") //中文转码有错误的函数。
        return enc.ConvertString(s)
    }
    func Server() {
        exit := make(chan bool)
        ip := net.ParseIP("127.0.0.1")
        addr := net.TCPAddr{ip, 8888, ""}
        go func() {
            listener, err := net.ListenTCP("tcp", &addr) //TCPListener listen
            if err != nil {
                fmt.Println("Initialize error", err.Error())
                exit <- true
                return
            }
            fmt.Println("Server listening...")
            tcpcon, err := listener.AcceptTCP() //TCPConn client
            if err != nil {
                fmt.Println(err.Error())
                //continue
            }
            fmt.Println("Client connect")
            data := make([]byte, 1024)
            if err != nil {
                fmt.Println("tcpcon.Read(data)" + err.Error())
            }
            //recv file name
            wc, err := tcpcon.Read(data)
            fo, err := os.Create("F:\\uploads\\" + string(data[0:wc]))
            if err != nil {
                fmt.Println("os.Create" + err.Error())
            }
            fmt.Println("the file's name is:", string(data[0:wc]))
            //recb file size
            wc, err = tcpcon.Read(data)
            fmt.Println("the file's size is:", string(data[0:wc]))
            defer fo.Close()
            for {
                c, err := tcpcon.Read(data) //???为何调用conn类的Read
                if err != nil {
                    fmt.Println("tcpcon.Read(data)" + err.Error())
                }
                if string(data[0:c]) == "filerecvend" {
                    fmt.Println("string(data[0:c]) == filerecvend is true")
                    tcpcon.Write([]byte("file recv finished!\r\n"))
                    tcpcon.Close()
                    break
                }
                //write to the file
                _, err = fo.Write(data[0:c])
                if err != nil {
                    fmt.Println("write err" + err.Error())
                }
            }
        }()
        <-exit
        fmt.Println(Show("server close!"))
    }
    func Client() {
        fmt.Println("send ur file to the destination", "input ur filename:")
        reader := bufio.NewReader(os.Stdin)
        input, _, _ := reader.ReadLine()
        fmt.Println(string(input))
        fi, err := os.Open(string(input))
        if err != nil {
            panic(err)
        }
        defer fi.Close()
        fiinfo, err := fi.Stat()
        fmt.Println("the size of file is ", fiinfo.Size(), "bytes") //fiinfo.Size() return int64 type
        conn, err := net.Dial("tcp", "127.0.0.1:8888")
        if err != nil {
            fmt.Println(Show("connect server fail！"), Show(err.Error()))
            return
        }
        defer conn.Close()
        //send filename
        _, err = conn.Write(input)
        if err != nil {
            fmt.Println("conn.Write", err.Error())
        }
        //send file size
        _, err = conn.Write([]byte(string(fiinfo.Size())))
        if err != nil {
            fmt.Println("conn.Write", err.Error())
        }
        buff := make([]byte, 1024)
        for {
            n, err := fi.Read(buff)
            if err != nil && err != io.EOF {
                panic(err)
            }
            if n == 0 {
                conn.Write([]byte("filerecvend"))
                fmt.Println("filerecvend")
                break
            }
            _, err = conn.Write(buff)
            if err != nil {
                fmt.Println(err.Error())
            }
        }
    }

