
---
date: 2016-12-31T11:33:02+08:00
title: "golang实现一个聊天室"
description: ""
disqus_identifier: 1485833582446432548
slug: "golang-shi-xian-yi-ge-liao-tian-shi"
source: "https://segmentfault.com/a/1190000007044082"
tags: 
- golang 
categories:
- 编程语言与开发
---

> 最近看了一下go语言，就试着写了一个聊天室，练练手而已，但是对于我一个搞php的来说，go语言对我启发很大。

**客服端**

    package main

    import (
        "fmt"
        "net"
        "os"
    )

    //定义通道
    var ch chan int = make(chan int)

    //定义昵称
    var nickname string

    func reader(conn *net.TCPConn) {
        buff := make([]byte, 128)
        for {
            j, err := conn.Read(buff)
            if err != nil {
                ch <- 1
                break
            }

            fmt.Println(string(buff[0:j]))
        }
    }

    func main() {

        tcpAddr, _ := net.ResolveTCPAddr("tcp", "127.0.0.1:9999")
        conn, err := net.DialTCP("tcp", nil, tcpAddr)

        if err != nil {
            fmt.Println("Server is not starting")
            os.Exit(0)
        }

        //为什么不能放到if之前？ err不为nil的话就是painc了 (painc 与 defer 辨析一下！！！)
        defer conn.Close()

        go reader(conn)

        fmt.Println("请输入昵称")

        fmt.Scanln(&nickname)

        fmt.Println("你的昵称为:", nickname)

        for {
            var msg string
            fmt.Scanln(&msg)
            b := []byte("<" + nickname + ">" + "说:" + msg)
            conn.Write(b)

            //select 为非阻塞的
            select {
            case <-ch:
                fmt.Println("Server错误!请重新连接!")
                os.Exit(1)
            default:
                //不加default的话，那么 <-ch 会阻塞for， 下一个输入就没有法进行
            }

        }

**服务器端**

    package main

    import (
        "fmt"
        "net"
    )

    var ConnMap map[string]*net.TCPConn

    func checkErr(err error) int {
        if err != nil {
            if err.Error() == "EOF" {
                //用户退出
                fmt.Println("用户推出了")
                return 0
            }
            fmt.Println("错误")
            return -1
        }
        return 1
    }

    func say(tcpConn *net.TCPConn) {
        for {
            //读取一个客户端发送过来的数据
            data := make([]byte, 128)
            total, err := tcpConn.Read(data)

            fmt.Println(string(data[:total]), err)

            flag := checkErr(err)
            if flag == 0 {
                //退出整个循环
                break
            }

            //广播形式，向各个客户端发送数据
            for _, conn := range ConnMap {
                if conn.RemoteAddr().String() == tcpConn.RemoteAddr().String() {
                    //不向数据输入的客户端发送消息
                    continue
                }
                conn.Write(data[:total])
            }
        }
    }

    func main() {
        tcpAddr, _ := net.ResolveTCPAddr("tcp", "127.0.0.1:9999")
        tcpListener, _ := net.ListenTCP("tcp", tcpAddr)
        /*
            map 定义完后，还要make? (哪些数据类型定义完后，还要make?)
            http://stackoverflow.com/questions/27267900/runtime-error-assignment-to-entry-in-nil-map
        */
        ConnMap = make(map[string]*net.TCPConn)

        for {

            tcpConn, _ := tcpListener.AcceptTCP()
            defer tcpConn.Close()

            ConnMap[tcpConn.RemoteAddr().String()] = tcpConn
            fmt.Println("连接的客服端信息:", tcpConn.RemoteAddr().String())

            go say(tcpConn)
        }
    }

