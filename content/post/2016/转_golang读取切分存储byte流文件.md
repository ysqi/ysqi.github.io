
---
date: 2016-12-31T11:33:23+08:00
title: "golang读取切分存储byte流文件"
description: ""
disqus_identifier: 1485833603794396671
slug: "golang-dou-qu-qie-fen-cun-chu-byteliu-wen-jian"
source: "https://segmentfault.com/a/1190000005875143"
tags: 
- 视频处理 
- 存储技术 
- 二进制 
- 流媒体 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main

    import (
        "fmt"
        "os"
        "time"
    )

    func check(e error) {
        if e != nil {
            panic(e)
        }
    }
    func cat(f *os.File) []byte {
        var payload []byte
        for {
            buf := make([]byte, 1024)
            switch nr, err := f.Read(buf[:]); true {
            case nr < 0:
                fmt.Fprintf(os.Stderr, "cat: error reading: %s\n", err.Error())
                os.Exit(1)
            case nr == 0: // EOF
                return payload
            case nr > 0:
                payload = append(payload, buf...)
            }
        }

    }

    func main() {
        file, err := os.Open("test.flv")
        if err != nil {
            panic(err)
        }
        defer file.Close()
        fmt.Println(file)
        payload := cat(file)
        fo, errs := os.Create(fmt.Sprintf("./%d.bmp", time.Now().UnixNano())) //time.Now().UnixNano()
        check(errs)
        _, errs = fo.Write(payload)

    }

