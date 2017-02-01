
---
date: 2016-12-31T11:32:46+08:00
title: "Golang流式解析Json"
description: ""
disqus_identifier: 1485833566493350197
slug: "Golang-liu-shi-jie-xi--Json"
source: "https://segmentfault.com/a/1190000007672560"
tags: 
- golang 
- json 
topics:
- 编程语言与开发
---

json-iterator
库：[`https://github.com/json-iterator/go`](https://github.com/json-iterator/go)

动机
====

现有的golang解析json的库都是push模式的，缺少一种基于pull
api的库。另外就是看一下golang解析json的速度到底如何，还有多少的提高空间。

API 风格
========

api 风格上是以 StAX 为基础，但是针对 JSON 做了特别的优化。比 StAX 和 SAX
都更简单可控。当然如果需要最简单，还是 DOM 类的 api
最简单。使用流式pull的api为的就是最大化控制解析过程。

解析 Array
----------

    iter := ParseString(`[1,2,3]`)
    for iter.ReadArray() {
      iter.ReadUint64()
    }

可以看到，pull api 的风格非常不同。整个解析流程是调用者驱动的

解析 Object
===========

    type TestObj struct {
        Field1 string
        Field2 uint64
    }

    iter := ParseString(`{"field1": "1", "field2": 2}`)
    obj := TestObj{}
    for field := iter.ReadObject(); field != ""; field = iter.ReadObject() {
        switch field {
        case "field1":
            obj.Field1 = iter.ReadString()
        case "field2":
            obj.Field2 = iter.ReadUint64()
        default:
            iter.ReportError("bind object", "unexpected field")
        }
    }

解析过程不依赖反射，解析出来的值干什么事情完全由你来操纵。你可以直接做一些累加操作，而不把值先绑定到对象上。

SKIP
----

    iter := ParseString(`[ {"a" : [{"b": "c"}], "d": 102 }, "b"]`)
    iter.ReadArray()
    iter.Skip()
    iter.ReadArray()
    if iter.ReadString() != "b" {
        t.FailNow()
    }

对于不关心的字段，可以选择跳过。

性能优化
========

这个项目的另外一个目的是看一下golang原生的json
api是快还是慢，有没有提高空间。

基于流解析，无需一次读到内存里
------------------------------

    // "encoding/json"
    func Benchmark_stardard_lib(b *testing.B) {
        b.ReportAllocs()
        for n := 0; n < b.N; n++ {
            file, _ := os.Open("/tmp/large-file.json")
            result := []struct{}{}
            decoder := json.NewDecoder(file)
            decoder.Decode(&result)
            file.Close()
        }
    }

5 215547514 ns/op 71467118 B/op 272476 allocs/op

    // "github.com/json-iterator/go"
    func Benchmark_jsoniter(b *testing.B) {
        b.ReportAllocs()
        for n := 0; n < b.N; n++ {
            file, _ := os.Open("/tmp/large-file.json")
            iter := jsoniter.Parse(file, 1024)
            for iter.ReadArray() {
                iter.Skip()
            }
            file.Close()
        }
    }

10 110209750 ns/op 4248 B/op 5 allocs/op

可以看到 json iterator
的实现对于内存占用非常节省。比标准库的实现快一倍。GC压力就小更多了。

直接解析出 int
--------------

对于 int 的解析无需两遍，一次性读取。把 ParseInt 的实现合并到 json
解析的代码里。

    func Benchmark_jsoniter_array(b *testing.B) {
        for n := 0; n < b.N; n++ {
            iter := ParseString(`[1,2,3]`)
            for iter.ReadArray() {
                iter.ReadUint64()
            }
        }
    }

10000000 189 ns/op

    func Benchmark_json_array(b *testing.B) {
        for n := 0; n < b.N; n++ {
            result := []interface{}{}
            json.Unmarshal([]byte(`[1,2,3]`), &result)
        }
    }

1000000 1327 ns/op

这个场景是 7x 的速度

无反射的，有schema的解析
------------------------

按照schema解析，减少if-else判断。直接赋值，无需经过反射

    type Level1 struct {
        Hello []Level2
    }

    type Level2 struct {
        World string
    }

    func Benchmark_jsoniter_nested(b *testing.B) {
        for n := 0; n < b.N; n++ {
            iter := ParseString(`{"hello": [{"world": "value1"}, {"world": "value2"}]}`)
            l1 := Level1{}
            for l1Field := iter.ReadObject(); l1Field != ""; l1Field = iter.ReadObject() {
                switch l1Field {
                case "hello":
                    l1.Hello = readLevel1Hello(iter)
                default:
                    iter.Skip()
                }
            }
        }
    }

    func readLevel1Hello(iter *Iterator) []Level2 {
        l2Array := make([]Level2, 0, 2)
        for iter.ReadArray() {
            l2 := Level2{}
            for l2Field := iter.ReadObject(); l2Field != ""; l2Field = iter.ReadObject() {
                switch l2Field {
                case "world":
                    l2.World = iter.ReadString()
                default:
                    iter.Skip()
                }
            }
            l2Array = append(l2Array, l2)
        }
        return l2Array
    }

2000000 640 ns/op

    func Benchmark_json_nested(b *testing.B) {
        for n := 0; n < b.N; n++ {
            l1 := Level1{}
            json.Unmarshal([]byte(`{"hello": [{"world": "value1"}, {"world": "value2"}]}`), &l1)
        }
    }

1000000 1816 ns/op

总结
====

golang 自带的 json
库其实性能很不错了。根据benchmark（[https://github.com/json-itera...](https://github.com/json-iterator/go-benchmark)）其实速度比其他的基于流的解析库还要快（[https://github.com/ugorji/go/...](https://github.com/ugorji/go/blob/master/codec/json.go)）。而这个库
[https://github.com/pquerna/ff...](https://github.com/pquerna/ffjson)
虽然号称更快，但是不支持流式解析（要求所有的\[\]byte都提前读入到内存里）。大部分情况下，就用golang自带的就足够好了，别瞎整一些其他的json解析库。

如果需要pull
api，或者需要额外的2x\~6x性能，可以考虑：[`https://github.com/json-iterator/go`](https://github.com/json-iterator/go)

