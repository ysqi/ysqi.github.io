
---
date: 2016-12-31T11:34:30+08:00
title: "go语言reflect包使用的几个场景"
description: ""
disqus_identifier: 1485833670374883739
slug: "goyu-yan-reflectbao-shi-yong-de-ji-ge-chang-jing"
source: "https://segmentfault.com/a/1190000002736013"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

reflect包的几个使用场景：

**1. 遍历结构体字段名（避免代码的硬编码）**\
**2. 调用结构体方法（自动映射）**\
**3. 获取结构体的tag标记的值（json/xml转换）**\
4. // @todo更多的使用场景

代码：

一、\$GOPATH/reflectusage/main.go:

    // reflect使用场景
    package main

    import (
        "errors"
        "fmt"
        "reflect"
        "reflectusage/config"
    )

    func main() {

        // 创建Conf实例
        conf := config.Conf{}

        opName := "create"
        conf.Op = &opName
        conf.Port = 3308

        fmt.Printf("conf.Port=%d\n\n", conf.Port)

        // 结构信息
        t := reflect.TypeOf(conf)
        // 值信息
        v := reflect.ValueOf(conf)

        printStructField(&t)

        callMethod(&v, "SayOp", []interface{}{" Db"})

        // panic: reflect: Call of unexported method
        //callMethod(&v, "getDbConf", []interface{}{})

        getTag(&t, "Op", "json")
        getTag(&t, "Op", "xml")
        getTag(&t, "nofield", "json")
    }

    // 场景1：遍历结构体字段名
    func printStructField(t *reflect.Type) {
        fieldNum := (*t).NumField()
        for i := 0; i < fieldNum; i++ {
            fmt.Printf("conf's field: %s\n", (*t).Field(i).Name)
        }
        fmt.Println("")
    }

    // 场景2：调用结构体方法
    func callMethod(v *reflect.Value, method string, params []interface{}) {
        // 字符串方法调用，且能找到实例conf的属性.Op
        f := (*v).MethodByName(method)
        if f.IsValid() {
            args := make([]reflect.Value, len(params))
            for k, param := range params {
                args[k] = reflect.ValueOf(param)
            }
            // 调用
            ret := f.Call(args)
            if ret[0].Kind() == reflect.String {
                fmt.Printf("%s Called result: %s\n", method, ret[0].String())
            }
        } else {
            fmt.Println("can't call " + method)
        }
        fmt.Println("")
    }

    // 场景3：获取结构体的tag标记
    func getTag(t *reflect.Type, field string, tagName string) {
        var (
            tagVal string
            err    error
        )
        fieldVal, ok := (*t).FieldByName(field)
        if ok {
            tagVal = fieldVal.Tag.Get(tagName)
        } else {
            err = errors.New("no field named:" + field)
        }

        fmt.Printf("get struct[%s] tag[%s]: %s, error:%v\n", field, tagName, tagVal, err)
        fmt.Println("")
    }

    // @todo更多的使用场景

二、\$GOPATH/reflectusage/config/config.go:

    package config

    type Db struct {
        Port int
        Host string
        pw   string
    }

    type Conf struct {
        Op       *string `json:"jsonop" xml:"xmlOpName"`
        Charlist *string
        Length   *int
        Num      *int
        Output   *string
        Input    *string
        hidden   *string
        Db
    }

    func (this Conf) SayOp(subname string) string {
        return *this.Op + subname
    }

    func (this Conf) getDbConf() Db {
        return this.Db
    }

三、输出：

    conf.Port=3308

    conf's field: Op
    conf's field: Charlist
    conf's field: Length
    conf's field: Num
    conf's field: Output
    conf's field: Input
    conf's field: hidden
    conf's field: Db

    SayOp Called result: create Db

    get struct[Op] tag[json]: jsonop, error:<nil>

    get struct[Op] tag[xml]: xmlOpName, error:<nil>

    get struct[nofield] tag[json]: , error:no field named:nofield

