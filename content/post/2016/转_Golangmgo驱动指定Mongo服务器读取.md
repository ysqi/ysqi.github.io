
---
date: 2016-12-31T11:35:01+08:00
title: "Golangmgo驱动指定Mongo服务器读取"
description: ""
disqus_identifier: 1485833701229669949
slug: "Golang-mgoqu-dong-zhi-ding-Mongofu-wu-qi-dou-qu"
source: "https://segmentfault.com/a/1190000000460489"
tags: 
- mongodb 
- mgo 
- golang 
categories:
- 编程语言与开发
---

Replica Sets搭建
================

服务器采用Replica Sets搭建，可参考[Deploy a Replica
Set](http://docs.mongodb.org/manual/tutorial/deploy-replica-set/)

读模式
======

Mongod的读模式共有五种：

-   primary. 在主节点上进行所有的读操作
-   primaryPreferred.
    优先在主节点上进行读操作，如果主节点不可用，再从从节点操作。
-   secondary.所有的读操作在从节点上进行。
-   secondaryPreferred.优先在从节点进行读操作，如果所有从节点都不可用，再从主节点操作。
-   nearest. 根据网络延迟时间 ，就近进行读操作，不考虑节点类型。

配置节点Tags Sets
=================

Tag sets 允许指定一个replica
set进行读操作，其中Mongod的读模式必须是以下四种之一：\
`primaryPreferred`、`secondary`、`secondaryPreferred`、`nearest`\
Tags Sets配置参考：[Configure Replica Set Tag
Sets](http://docs.mongodb.org/manual/tutorial/configure-replica-set-tag-sets)\
主要操作如下：

    conf = rs.conf()
    conf.members[0].tags = { "dc": "east", "use": "production"  }
    conf.members[1].tags = { "dc": "east", "use": "reporting"  }
    conf.members[2].tags = { "use": "production"  }
    rs.reconfig(conf)

mgo代码示例
===========

根据以上的配置，如果需要指定从members
1中进行数据库读操作，可采取以下代码：

    session, err := mgo.Dial("localhost")
    if err != nil {
        log.Fatalln(err)
    }
    defer session.Close()
    session.SetMode(mgo.Eventual, true) //需要指定为Eventual
    session.SelectServers(bson.D{{"dc", "east"}, {"use", "reporting"}}) // 指定从1中读取
    db := session.DB("test")
    col := db.C("tbl")
    data := make([]interface{}, 10)
    col.Find(nil).Limit(10).All(&data)
    log.Println(data)

