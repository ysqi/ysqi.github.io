
---
date: 2016-12-31T11:32:43+08:00
title: "golangjson解析器哪家强？"
description: ""
disqus_identifier: 1485833563800292972
slug: "golang-json-jie-xi-qi-na-jia-jiang-？"
source: "https://segmentfault.com/a/1190000007721025"
tags: 
- golang 
- json 
categories:
- 编程语言与开发
---

全文链接：
[`https://github.com/json-iterator/go-benchmark`](https://github.com/json-iterator/go-benchmark)

目的不是推销 json-iterator 。而是证明 json-iterator
不比其他的库更慢，从而使得大家可以把吐槽点放到其他方面：比如特性是不是齐全，
api 是不是友好。重新发明 json 解析器是因为经常需要处理奇怪格式的 json
，而又不想把数据转两遍。市面上没有 api
满足我的需求的，后面我会专门写一篇 api 介绍的文章来演示 json-iterator
的独特性。（
[https://github.com/json-itera...](https://github.com/json-iterator/go/blob/master/README.md)
）

-   jsonparser: `https://github.com/buger/jsonparser`

-   jsoniter pull-api:
    [`https://github.com/json-iterator/go`](https://github.com/json-iterator/go)

-   jsoniter reflect-api:
    `https://github.com/json-iterator/go/blob/master/jsoniter_reflect.go`

-   encoding/json: golang standard lib

-   easy json: `https://github.com/mailru/easyjson`

测试设备

-   CPU: i7-6700K @ 4.0G

-   Level 1 cache size: 4 x 32 KB 8-way set associative instruction
    caches

-   Level 2 cache size: 4 x 256 KB 4-way set associative caches

-   Level 3 cache size: 8 MB 16-way set associative shared cache

-   Go: 1.8beta1

small payload
=============

[https://github.com/json-itera...](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_small_payload_test.go)

  jsonparser    jsoniter pull-api   jsoniter reflect-api   encoding/json   easyjson
  ------------- ------------------- ---------------------- --------------- -------------
  599 ns/op     515 ns/op           684 ns/op              2453 ns/op      687 ns/op
  64 B/op       64 B/op             256 B/op               864 B/op        64 B/op
  2 allocs/op   2 allocs/op         4 allocs/op            31 allocs/op    2 allocs/op

encoding/json 在 i7-6700K 上性能还不错，但是在缓存小一些的 cpu
上性能要比这慢更多。

medium payload
==============

  jsonparser    jsoniter pull-api   jsoniter reflect-api   encoding/json   easyjson
  ------------- ------------------- ---------------------- --------------- -------------
  5238 ns/op    4111 ns/op          4708 ns/op             24939 ns/op     7361 ns/op
  104 B/op      104 B/op            368 B/op               808 B/op        248 B/op
  4 allocs/op   4 allocs/op         14 allocs/op           18 allocs/op    8 allocs/op

[https://github.com/json-itera...](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_medium_payload_test.go)

json-iterator 的反射 api 也是相当快的。

large payload
=============

[https://github.com/json-itera...](https://github.com/json-iterator/go-benchmark/blob/master/src/github.com/json-iterator/go-benchmark/benchmark_large_payload_test.go)

  jsonparser    jsoniter pull-api   encoding/json
  ------------- ------------------- ---------------
  38334 ns/op   38463 ns/op         290778 ns/op
  0 B/op        0 B/op              2128 B/op
  0 allocs/op   0 allocs/op         46 allocs/op

jsonparser 在大部分字段不使用的时候，要快那么一丁点。

large file
==========

test file used:
[https://github.com/json-itera...](https://github.com/json-iterator/test-data/blob/master/large-file.json)

  jsonparser       jsoniter pull-api   encoding/json
  ---------------- ------------------- ------------------
  42698634 ns/op   37760014 ns/op      235354502 ns/op
  67107104 B/op    4248 B/op           71467896 B/op
  19 allocs/op     5 allocs/op         272477 allocs/op

jsonparser 等其他一大票 json 解析器都是以 \[\]byte
作为输入的，简直是为跑分而生。关于这一点 jackson 的作者也有吐槽（
[https://www.infoq.com/news/20...](https://www.infoq.com/news/2014/05/jackson-founder-responds)
）。而 jsoniter 可以支持 io.Reader 作为输入，对于大文件处理非常友好。



