
---
date: 2017-05-24T09:17:27+08:00
title: "谈谈一些关于mgo的用法"
description: ""
disqus_identifier: 1495588647322916309
slug: "tan-tan-yi-xie-guan-yu-mgode-yong-fa"
source: "https://segmentfault.com/a/1190000009370421"
tags: 
- mongodb 
- golang 
topics:
- 编程语言与开发
---

 前言
=====

最近在项目中使用mongodb进行简单的数据分析，在使用mongodb驱动mgo时遇到一些问题，比如在mongodb中执行命令成功，到了mgo中就执行失败。在这里谈一谈实践过程中遇到的问题，基础的用法不再说明了，可以自行百度。

使用
====

查找(Find)
----------

这个估计是mongodb里用的最多的了吧，mgo中使用Find(query
interface{})，query参数一般传入[]bson.M。下面给个例子：

    m := bson.M{
            "CurTimestamp": bson.M{
                "$gte": start,
                "$lte": end,
            },
            "Account":    account,
            "ToNodeType": "cloud",
        }
        session.DB("db").C("collect").Find(m).Count()

这里查找时间戳内，账号为account，节点类型为cloud的数据并统计个数。

聚合管道（Aggregation Pipeline）
--------------------------------

聚合管道在mgo中为Pipe(pipeline interface{})
，这个和bash中使用的管道很像，数据可以被层层处理。一般传入的参数为[]bson.M。这个[]bson.M里如果还有嵌套则还要使用[]bson.M(刚开始使用就被坑了一下)。比如这里首先匹配节点类型和账号，时间戳在一段时间内，然后根据名字分组统计数量，最后排序取最前面的三个。

       //这个就可以传入Pipe
       m := []bson.M{
            {"$match": bson.M{"NodeType": "cloud", "Account": account, "CurTimestamp": bson.M{"$gte": start, "$lte": end}}},
            {"$group": bson.M{"_id": "$TagName", "count": bson.M{"$sum": 1}}},
            {"$sort": bson.M{"count": -1}},
            {"$limit": 3},
        }
        //这里就可以取到输出的数据
        var values []result
        session.DB("db").C("collect").Pipe(m).All(&values)

数据是一层一层过滤下来的。当然mongodb中的聚合命令不止这些，用法大同小异

### MapReduce

如果要实现一些高级功能，mongodb的基本命令满足不了你，可能就要使用这个了。mongodb中要实现MapReduce就要实现Map函数和Reduce函数，Map函数调用emit将key和value传给Reduce函数处理。这里给的例子首先计时间戳在哪个时间范围内，然后累加这个值，js不是很精通，写的不好请见谅。

    m := new(mgo.MapReduce)
        m.Map = `function() { var date = new Date();
        date.setTime(this.CurTimestamp / 1000);
        var hour = date.getHours();
        if((hour >= 6) && (hour <= 11)) {
            result.morning++;
        }else if((hour >= 12) && (hour <= 18)){
            result.afternoon ++;
        }else if((hour >= 19) && (hour <= 23)) {
            result.night ++;
        }else{
            result.am ++;
        }
        emit(this.Account, {});}`
        m.Reduce = `function() {return result;}`
        m.Scope = bson.M{
            "result": bson.M{
                "morning":   0,
                "afternoon": 0,
                "night":     0,
                "am":        0,
            },
        }
        var value []timeResult
        session.DB("db").C("collect").Find().MapReduce(m, &value)

这里的map函数负责计算时间戳范围，result是引入的外部变量。这里就可以计算出这个账号产生数据的时间范围统计。

结语
====

其实用多了以后就基本能熟练使用了，重要还是了解mogodb命令的使用。

