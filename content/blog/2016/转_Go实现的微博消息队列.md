
---
date: 2016-12-31T11:33:54+08:00
title: "Go实现的微博消息队列"
description: ""
disqus_identifier: 1485833634357399585
slug: "Goshi-xian-de-wei-bo-xiao-xi-dui-lie"
source: "https://segmentfault.com/a/1190000004677575"
tags: 
- golang 
topics:
- 编程语言与开发
---

有兴趣看实现消息队列原理的，请移步
<https://github.com/YoungPioneers/mgq>，感谢您的宝贵意见

Introduction
============

Memcached Go Queue, 简称mgq,
是一个用[Go](https://golang.org)语言写的，基于memcached协议的消息队列。其父亲[mcq](https://github.com/stvchu/memcacheq.git)是最早应用于[weibo](http://weibo.com)的基础消息中间件，有着高性能，解耦的优点，使得其广泛应用于微博

Features
========

mgq是一个基于NOSQL数据库[BerkeleyDB](http://www.oracle.com/technetwork/cn/database/database-technologies/berkeleydb/overview/index.html)写的FIFO消息队列，目前支持的特性如下：

-   一写多读：举个例子，set myqueue 'one
    message'，只要get的时候，myqueue开头，\#分隔，如myqueue\#1，多个客户端之间读是彼此独立的，是不受影响的

-   默认的get是读取队列中未读取的最旧消息

-   支持getc操作，支持获取队列中某一个cursor位置的数据：举个例子，假设myqueue已经有1000条数据，getc
    myqueue 99,就可以获取队列当中cursor为99的消息

-   支持getr操作，支持获取队列中某一个start
    cursor位置开始，到end的数据：举个例子，假设myqueue已经有20条数据，getr
    myqueue 1 10 ,就可以获取队列当中cursor为\[1-10\]的消息 「内测中」

-   getn支持timeout机制的阻塞api来获取队列中的最新消息，举个例子：getn
    queue
    10,意味着10s内有数据则立马返回，否则会10s后立马返回数据不存在的错误，默认getn的timeout是0s，永不超时（需要注意的是如果客户端有getn的操作，则set的另一个客户端需要调用setn）

Benchmark
=========

针对消息的丢失率，做了一下单个set和get的测试，下面的是消息数为30w，50w，100w时候的结果

message total:30w

    mgq message set total:300000, cost total time:123518222205 ns, 2428 per/s ,fail set total:0
    mgq message get total:300000, cost total time:51103619703 ns, 5870 per/s ,fail get total:0

message total:50w

    mgq message set total:500000, cost total time:210480212729 ns, 2375 per/s ,fail set total:0
    mgq message get total:500000, cost total time:87694742059 ns, 5701 per/s ,fail get total:0

message total:100w

    mgq message set total:1000000, cost total time:422339921379 ns, 2367 per/s ,fail set total:0
    mgq message get total:1000000, cost total time:173768683759 ns, 5754 per/s ,fail get total:0

同时简单做了一下压测，下面的结果依次是1，2，3，4个routine，单个set和get的相对平均耗时时间,[Benchmark
code](https://github.com/YoungPioneers/mgq/benchmark/mgqPerformance_test.go)

    Benchmark_MgqMultiSetAndGet-4         2000        546617 ns/op (1829 per/s)
    Benchmark_MgqMultiSetAndGet-4         2000        583259 ns/op (1714 per/s)
    Benchmark_MgqMultiSetAndGet-4         2000        723603 ns/op (1381 per/s)
    Benchmark_MgqMultiSetAndGet-4         2000        754741 ns/op (1324 per/s)

