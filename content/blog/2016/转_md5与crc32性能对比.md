
---
date: 2016-12-31T11:34:06+08:00
title: "md5与crc32性能对比"
description: ""
disqus_identifier: 1485833646435370991
slug: "md5yu-crc32xing-neng-dui-bi"
source: "https://segmentfault.com/a/1190000004000838"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

感觉MD5算法复杂度比crc32高很多，具体高多少呢？\
测试一下

    // main.go
    package main

    import (
        "crypto/md5"
        "fmt"
        "hash/crc32"
    )

    func main() {

        data := []byte("test")
        fmt.Printf("%x", md5.Sum(data))

    }

    func Crc32IEEE(data []byte) uint32 {
        return crc32.ChecksumIEEE(data)
    }

    func Md5(data []byte) [16]byte {
        return md5.Sum(data)
    }

    // main_test.go
    package main

    import "testing"

    func BenchmarkCrc32(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Crc32IEEE([]byte("test"))
        }
    }

    func BenchmarkMd5(b *testing.B) {
        for i := 0; i < b.N; i++ {
            Md5([]byte("test"))
        }
    }

    go test -bench=.

------------------------------------------------------------------------

PASS\
BenchmarkCrc32-4 20000000 64.9 ns/op\
BenchmarkMd5-4 5000000 274 ns/op\
ok test 3.022s

------------------------------------------------------------------------

md5大致慢4倍左右

