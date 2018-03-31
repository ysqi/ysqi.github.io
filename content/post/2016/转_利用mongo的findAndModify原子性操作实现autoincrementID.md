
---
date: 2016-12-31T11:33:44+08:00
title: "利用mongo的findAndModify原子性操作实现autoincrementID"
description: ""
disqus_identifier: 1485833624956065072
slug: "li-yong-mongode-findAndModifyyuan-zi-xing-cao-zuo-shi-xian-auto-increment-ID"
source: "https://segmentfault.com/a/1190000005026931"
tags: 
- golang 
- mongodb 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

### 实际情况

默认情况下，mongo使用`_id`自动生成uniq
id，而且在mongo自带的命令里，无法指定一个自增字段。自增字段在多线程时必须是原子性的，这在大数据情况下很难实现伸缩性(scalability)。

> Generally in MongoDB, you would not use an auto-increment pattern for\
> the \_id field, or any field, because it does not scale for databases\
> with large numbers of documents. Typically the default value ObjectId\
> is more ideal for the \_id.

不过好消息是，`_id`不一定非得是ObjectId类型，它可以是任何类型。而且，在mongo里面，有一个findAndModify命令是原子性的。我们可以使用它实现auto
increment ID。\
在golang的mgo库里，`Apply`命令可以实现`findAndModify`功能

### 具体策略和代码

#### 策略

假设我们要把test表的`_id`字段设置为自增的，我们需要先创建一个`counter`表，这个表里面保存一个`seq`字段，通过在`seq`上使用Apply，获取新doc的seq值作为test表中下一个id的值。

#### 代码如下：

    package main

    import (
        "fmt"
        "gopkg.in/mgo.v2"
        "gopkg.in/mgo.v2/bson"
        "log"
        "math/rand"
        "sync"
    )

    func init() {
        log.SetFlags(log.Lshortfile | log.LstdFlags)
    }
    func main() {
        session, err := mgo.Dial("usr:pwd@127.0.0.1:27017/dbname")
        if err != nil {
            log.Fatal("无法打开MongoDB！")
            return
        }
        defer session.Close()

        clt := session.DB("dbname").C("test")
        getNextSeq := func() func() int {
            counter := session.DB("dbname").C("counter")
            cid := "counterid"
            return func() int {
                change := mgo.Change{
                    Update:    bson.M{"$inc": bson.M{"seq": 1}},
                    Upsert:    true,
                    ReturnNew: true,
                }
                doc := struct{ Seq int }{}
                if _, err := counter.Find(bson.M{"_id": cid}).Apply(change, &doc); err != nil {
                    panic(fmt.Errorf("get counter failed:", err.Error()))
                }
                log.Println("seq:", doc)
                return doc.Seq
            }
        }()

        wg := sync.WaitGroup{}
        // 创建10个 go routine模拟多线程环境
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go func(i int) {
                _row := map[string]interface{}{
                    "_id":       getNextSeq(),
                    "username":  fmt.Sprintf("name%2d", i),
                    "telephone": fmt.Sprintf("tel%4d", rand.Int63n(1000))}
                if err := clt.Insert(_row); err != nil {
                    log.Printf("insert %v failed: %v\n", _row, err)
                } else {
                    log.Printf("insert: %v success!\n", _row)
                }
                wg.Done()
            }(i)
        }
        wg.Wait()
    }

#### 输出：

    2016/04/28 17:05:28 main.go:37: seq: {201}
    2016/04/28 17:05:28 main.go:54: insert: map[_id:201 username:name 9 telephone:tel 410] success!
    2016/04/28 17:05:28 main.go:37: seq: {204}
    2016/04/28 17:05:28 main.go:37: seq: {203}
    2016/04/28 17:05:28 main.go:37: seq: {202}
    2016/04/28 17:05:28 main.go:37: seq: {206}
    2016/04/28 17:05:28 main.go:37: seq: {205}
    2016/04/28 17:05:28 main.go:54: insert: map[_id:202 username:name 5 telephone:tel  51] success!
    2016/04/28 17:05:28 main.go:54: insert: map[_id:204 username:name 2 telephone:tel 551] success!
    2016/04/28 17:05:28 main.go:54: insert: map[telephone:tel 821 _id:203 username:name 1] success!
    2016/04/28 17:05:28 main.go:37: seq: {207}
    2016/04/28 17:05:28 main.go:54: insert: map[_id:205 username:name 6 telephone:tel 320] success!
    2016/04/28 17:05:28 main.go:54: insert: map[_id:206 username:name 4 telephone:tel 937] success!
    2016/04/28 17:05:28 main.go:54: insert: map[_id:207 username:name 3 telephone:tel 758] success!
    2016/04/28 17:05:28 main.go:37: seq: {208}
    2016/04/28 17:05:28 main.go:54: insert: map[username:name 7 telephone:tel 148 _id:208] success!
    2016/04/28 17:05:28 main.go:37: seq: {209}
    2016/04/28 17:05:28 main.go:54: insert: map[_id:209 username:name 0 telephone:tel 216] success!
    2016/04/28 17:05:28 main.go:37: seq: {210}
    2016/04/28 17:05:28 main.go:54: insert: map[_id:210 username:name 8 telephone:tel 449] success!

#### mongo查询结果

    db.test.find({_id:{$gt:200}})
    { "_id" : 201, "username" : "name 9", "telephone" : "tel 410" }
    { "_id" : 202, "username" : "name 5", "telephone" : "tel  51" }
    { "_id" : 203, "username" : "name 1", "telephone" : "tel 821" }
    { "_id" : 204, "username" : "name 2", "telephone" : "tel 551" }
    { "_id" : 205, "username" : "name 6", "telephone" : "tel 320" }
    { "_id" : 206, "username" : "name 4", "telephone" : "tel 937" }
    { "_id" : 207, "username" : "name 3", "telephone" : "tel 758" }
    { "_id" : 208, "username" : "name 7", "telephone" : "tel 148" }
    { "_id" : 209, "username" : "name 0", "telephone" : "tel 216" }
    { "_id" : 210, "username" : "name 8", "telephone" : "tel 449" }

