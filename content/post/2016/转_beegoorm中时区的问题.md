
---
date: 2016-12-31T11:32:27+08:00
title: "beegoorm中时区的问题"
description: ""
disqus_identifier: 1485833547213401563
slug: "beego-ormzhong-shi-ou-de-wen-ti"
source: "https://segmentfault.com/a/1190000008204928"
tags: 
- beego 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

先看简化后代码，下面只列出main函数

    func main() {
        t := "2017-01-19 00:00:00"
        o := orm.NewOrm()

        qb, _ := orm.NewQueryBuilder("mysql")
        sql := qb.Select("COUNT(*)").From("test").Where("create_time > ?").String()
        o.Raw(sql, t).Exec()

        o.QueryTable("test").Filter("create_time__gt", t).Count()
    }

这么看的话感觉两个SQL应该是相同的：

    [ORM] - 2017-01-19 19:28:02 - [Queries/default] - [  OK /     db.Exec /     1.2ms] - [SELECT COUNT(*) FROM test WHERE create_time > ?] - `2017-01-19 00:00:00`
    [ORM] - 2017-01-19 19:28:02 - [Queries/default] - [  OK / db.QueryRow /     2.3ms] - [SELECT COUNT(*) FROM `test` T0 WHERE T0.`create_time` > ? ] - `2017-01-19 00:00:00`

我在本机测试OK，但在另一个环境SQL是这样的：

    [ORM] - 2017-01-19 11:30:43 - [Queries/default] - [  OK /     db.Exec /     1.2ms] - [SELECT COUNT(*) FROM test WHERE create_time > ?] - `2017-01-19 00:00:00`
    [ORM] - 2017-01-19 11:30:43 - [Queries/default] - [  OK / db.QueryRow /     1.2ms] - [SELECT COUNT(*) FROM `test` T0 WHERE T0.`create_time` > ? ] - `2017-01-19 08:00:00`

相差8小时，第一时间想到时区问题，去有问题的环境一看果真如此。

然后看了下beego orm的代码，下面列出关键部分。\
1.`orm/db_utils.go`的`getFlatParams()`\
此函数是解析`Filter()`生成SQL的关键部分，如果`Filter()`第一个参数类型是Date或Datetime，第二个参数类型是string就把string解析成time.Time类型\
在上面case中`len(v) = 19`，执行`time.ParseInLocation(formatDateTime, s, DefaultTimeLoc)`，因为有问题的环境是UTC时区，所以此函数会把字符串`2017-01-19 00:00:00`解析成time.Time`2017-01-19 00:00:00 +0000 UTC`（变量t，但实际是东八区的时间，正确的t应该是`2017-01-19 00:00:00 +0800 CST`）。

    func getFlatParams(fi *fieldInfo, args []interface{}, tz *time.Location) (params []interface{}) {
    ……
        switch kind {
        case reflect.String:
            v := val.String()
            if fi != nil {
                if fi.fieldType == TypeDateField || fi.fieldType == TypeDateTimeField {
                    var t time.Time
                    var err error
                    if len(v) >= 19 {
                        s := v[:19]
                        t, err = time.ParseInLocation(formatDateTime, s, DefaultTimeLoc)
                    } else {
                        s := v
                        if len(v) > 10 {
                            s = v[:10]
                        }
                        t, err = time.ParseInLocation(formatDate, s, tz)
                    }
                    if err == nil {
                        if fi.fieldType == TypeDateField {
                            v = t.In(tz).Format(formatDate)
                        } else {
                            v = t.In(tz).Format(formatDateTime)
                        }
                    }
                }
            }
            arg = v
    ……
    }

2.`t.In(tz).Format(formatDateTime)`再次将t格式化为字符串，参数`tz`是关键，它是在下面代码中赋值的。因为MySQL设置的是东八区，所以会设置al.TZ为东八区，也就是`t.In(tz).Format(formatDateTime)`中tz是东八区，导致`Format`返回的字符串是`2017-01-19 08:00:00`，于是就有了上面两条SQL不同的问题。

    func detectTZ(al *alias) {
        // orm timezone system match database
        // default use Local
        al.TZ = time.Local
        ...... 
        switch al.Driver {
        case DRMySQL:
            row := al.DB.QueryRow("SELECT TIMEDIFF(NOW(), UTC_TIMESTAMP)")
            var tz string
            row.Scan(&tz)
            if len(tz) >= 8 {
                if tz[0] != '-' {
                    tz = "+" + tz
                }
                t, err := time.Parse("-07:00:00", tz)
                if err == nil {
                    al.TZ = t.Location()
                } else {
                    DebugLog.Printf("Detect DB timezone: %s %s\n", tz, err.Error())
                }
            }
    ......

