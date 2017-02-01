
---
date: 2016-12-31T11:33:45+08:00
title: "golang--mgo解析各种数据类型分析"
description: ""
disqus_identifier: 1485833625185938766
slug: "golang----mgojie-xi-ge-chong-shu-ju-lei-xing-fen-xi"
source: "https://segmentfault.com/a/1190000005026413"
tags: 
- mongodb 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main

    import (
        "gopkg.in/mgo.v2"
        "log"
        "reflect"
    )

    func main() {
        session, err := mgo.Dial("usr:pwd@127.0.0.1:27017/dbname")
        if err != nil {
            log.Fatal("无法打开MongoDB！")
            return
        }
        defer session.Close()

        clt := session.DB("mydb").C("userinfo")
        type row struct {
            UserName, 
            Telephone string
        }

        var _row, _row2 interface{}
        // _row和_row2 都是interface，也就是说它们可以指向任意类型，此时是指向row结构的指针
        _row = new(row)
        _row2 = new(row)
        // _row3 是具体struct
        _row3 := row{}
        
        log.Printf("type of &row: %v\n", reflect.TypeOf(&_row))
        log.Printf("type of row:  %v\n", reflect.TypeOf(_row2))
        log.Printf("type of &row3: %v\n", reflect.TypeOf(&_row3))

        it := clt.Find(nil).Limit(1).Iter()
        log.Println("test unmarshal using", reflect.TypeOf(&_row))
        for it.Next(&_row) {
            // 此时&_row是指针，指向的类型是interface 
            // mongo会把row当成map对待, 所有域的信息都会输出来
            log.Println(_row)
        }
        
        it = clt.Find(nil).Limit(1).Iter()
        log.Println("test unmarshal using", reflect.TypeOf(_row2))
        for it.Next(_row2) {
            // 此时row2是指向row结构的指针
            // 只有确定了类型后，才会当成具体类型
            log.Println(_row2)
        }

        it = clt.Find(nil).Limit(1).Iter()
        log.Println("test unmarshal using", reflect.TypeOf(&_row3))
        for it.Next(&_row3) {
            // 只有确定了类型后，才会当成具体类型
            log.Println(_row3)
        }
    }

输出结果如下：

    2016/04/28 16:23:22 type of &row: *interface {}
    2016/04/28 16:23:22 type of row:  *main.row
    2016/04/28 16:23:22 type of &row3: *main.row
    2016/04/28 16:23:22 test unmarshal using *interface {}
    2016/04/28 16:23:22 map[_id:ObjectIdHex("5721c757a8917850b210f0ca") username:xiaoming telephone:2222 address:beijing]
    2016/04/28 16:23:22 test unmarshal using *main.row
    2016/04/28 16:23:22 &{xiaoming  2222}
    2016/04/28 16:23:22 test unmarshal using *main.row
    2016/04/28 16:23:22 {xiaoming  2222}

