
---
date: 2016-12-31T11:34:10+08:00
title: "Go操作Redis"
description: ""
disqus_identifier: 1485833650837894138
slug: "Gocao-zuo-Redis"
source: "https://segmentfault.com/a/1190000003943281"
tags: 
- redis 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

Start
-----

在SF文章中，关于PHP，python操作redis的文章已经很多了。可是少了go对redis的操作。最近也是刚开始学Go,由于对redis的偏爱，也顺便学习了一下，并总结下来。

Go-Redis包管理
--------------

-   很明显，我们Go是没有这个包的，那么我们可以利用GO的命令。首先我们得先配置`GOPATH`的路径，我选择的是/data/go/goSource作为我包的路径，所以shell执行如下

`export GOPATH=/data/go/goSource`

-   `go get github.com/alphazero/Go-Redis`这样就载下了Go-Redis包了

代码验证
--------

-   确认redis服务已经开启

-   redis.conf注意bind ip 确认可以被连接

-   code

<!-- -->

    package main
    import (
        "fmt"
        "github.com/alphazero/Go-Redis"
    )

    func main() {
        //DefaultSpec()创建一个连接
        //选择host,若需要auth,则password填写
        //spec        := redis.DefaultSpec().Host("192.168.1.111").Db(0).Password("");
        //若连接的本机redis-server,则host可以省略
        spec        := redis.DefaultSpec().Db(0).Password("");
        client, err := redis.NewSynchClientWithSpec (spec);
        
        if err != nil {
            fmt.Println("Connect redis server fail");
            return
        }

        dbkey := "test";
        value :=[]byte("Hello world!");
        client.Set(dbkey, value);
        
        getValue ,err:= client.Get(dbkey);
        if err != nil {
            fmt.Println("Get Key fail");
            return
        } else {
            str := string(getValue);
            fmt.Println(str);
        }
        
    }

总结
----

-   我们不难发现，存到redis，是byte,取的值也是byte。用的时候，需要做相关转换。

-   当然今天写的比较少，只是基础的String
    ，其他数据类型操作，比如hash，`client.Hset(dbkey, property, value)`.操作方法和其他语言一致。`注意首字母大写`



