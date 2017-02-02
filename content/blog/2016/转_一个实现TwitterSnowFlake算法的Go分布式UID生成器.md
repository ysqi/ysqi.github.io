
---
date: 2016-12-31T11:33:55+08:00
title: "一个实现TwitterSnowFlake算法的Go分布式UID生成器"
description: ""
disqus_identifier: 1485833635697591336
slug: "yi-ge-shi-xian--Twitter-SnowFlake-suan-fa--de--Go-fen-bu-shi--UID-sheng-cheng-qi"
source: "https://segmentfault.com/a/1190000004548112"
tags: 
- twitter 
- 线程安全 
- 分布式系统 
- golang 
topics:
- 编程语言与开发
---

goSnowFlake
===========

[](https://travis-ci.org/zheng-ji/goSnowFlake)\
[](https://godoc.org/github.com/zheng-ji/goSnowFlake)

根据 Twitter SnowFlake 算法, 实现的分布式线程安全 UID 生成器

Feature
-------

-   线程安全的 UID 生成器

-   绿色可插拔，无需依赖 Redis,Mysql,无状态

-   适合分布式系统

-   实现 Twitter SnowFlake 理论

Description
-----------

    0               41                 51                   64
    +---------------+----------------+-----------+
    |timestamp(ms)  | worker node id | sequence      |
    +---------------+----------------+-----------+

    id  = timestamp | workerid | sequence (eg. 1451063443347648410)

由三部分与运算组合而成，分别是毫秒级别的时间戳，机器 workerid,
以及为了解决冲突的序列号

Installation
------------

    go get github.com/zheng-ji/goSnowFlake

Example
-------
```Go
    import (
            "fmt"
            "github.com/zheng-ji/goSnowFlake"
    )

    func main() {
        // Params: Given the workerId, 0 < workerId < 1024
            iw, err := goSnowFlake.NewIdWorker(1)
            if err!= nil {
                    fmt.Println(err)
            }
            for i := 0; i < 100; i++ {
                    if id, err := iw.NextId(); err != nill {
                            fmt.Println(id)
            }
            }
    }
```
Documentation
-------------

-   [Twitter Blog
    Reference](https://blog.twitter.com/2010/announcing-snowflake)

-   [Reddit
    Discuss](https://www.reddit.com/comments/cajap/twitter_announces_snowflake_a_distributed_unique/)

License
-------

Copyright (c) 2015 by [zheng-ji](http://zheng-ji.info) released under
MIT License.

