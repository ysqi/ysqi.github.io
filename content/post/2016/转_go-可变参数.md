
---
date: 2016-12-31T11:33:26+08:00
title: "go-可变参数"
description: ""
disqus_identifier: 1485833606909730767
slug: "go-ke-bian-can-shu"
source: "https://segmentfault.com/a/1190000005749017"
tags: 
- 参数 
- interface 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

今天在尝试用go写一个简单的orm的时候 发现
在调用可变参数函数时，不是总能使用省略号将一个切片展开，有时候编译器可能会报错
再此用几个简单的例子作为说明

当不太确定数据类型的时候我们通常采用空接口

    tests1(789)
    fmt.Println("-------------")
    tests1("789")

    func tests1(arg interface{}) {
        fmt.Println("value:", arg)
        fmt.Println("type:", reflect.TypeOf(arg).Name())
    }

输出结果

    value: 789
    type: int
    -------------
    value: 789
    type: string

在使用相同类型的可变入参时

    tests([]string{"4", "5", "6"}...)

    func tests(args ...string) {
        for i, v := range args {
            fmt.Println(i, "----", v)
        }
    }

输出结果

    0 ---- 4
    1 ---- 5
    2 ---- 6

当使用interface{}作为可变入参时

    func testParams(args ...interface{}) {
        for i, v := range args {
            if s, ok := v.(string); ok {
                fmt.Println("----", s)
            }
            if s, ok := v.([]string); ok {
                for i, v := range s {
                    fmt.Println(i, "[]----", v)
                }
            }
            fmt.Println(i, v)
        }
    }

出现错误

    cannot use []string literal (type []string) as type []interface {} in argument to testParams      

当看到这里时候答案已经露出水面了\
这里提供两种解决方案

第一种方法

    s := []string{"4", "5", "6"}
    var d []interface{} = []interface{}{s[0], s[1], s[2]}
    testParams(d...)

结果

    ---- 4
    0 4
    ---- 5
    1 5
    ---- 6
    2 6

第二种方法

    s := []string{"4", "5", "6"}
    var d []interface{}
    d = append(d, s)
    testParams(d...)

结果

    0 []---- 4
    1 []---- 5
    2 []---- 6
    0 [4 5 6]

总结： 在使用interface{}作为可变入参时 传入的参数要做类型转换

