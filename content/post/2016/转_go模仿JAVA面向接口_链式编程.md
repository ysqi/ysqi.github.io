
---
date: 2016-12-31T11:34:30+08:00
title: "go模仿JAVA面向接口_链式编程"
description: ""
disqus_identifier: 1485833670156200182
slug: "go-mo-fang-JAVAmian-xiang-jie-kou-_lian-shi-bian-cheng"
source: "https://segmentfault.com/a/1190000002737864"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

Test.go

    package test

    func NewTest() ITestIntf {
        return &test{""}
    }

    type ITestIntf interface {
        GetName() string
        SetName(string) ITestIntf
    }

    type test struct {
        name string
    }

    func (t *test) GetName() string {
        return (*t).name
    }

    func (t *test) SetName(n string) ITestIntf {
        (*t).name =n
        return t
    }

SubTest.go

    package test

    func NewSubTest() ISubTestIntf {
        return &subTest{NewTest(), "", 0}
    }

    type ISubTestIntf interface {
        ITestIntf
        GetSex() string
        SetSex(string) ISubTestIntf
        GetAge() int
        SetAge(int) ISubTestIntf
    }

    func (t *subTest) GetSex() string {
        return (*t).sex
    }

    func (t *subTest) SetSex(s string) ISubTestIntf {
        (*t).sex =s
        return t
    }

    func (t *subTest) GetAge() int {
        return (*t).age
    }

    func (t *subTest) SetAge(a int) ISubTestIntf {
        (*t).age =a
        return t
    }

chan.go

    import (
        "fmt"
        "test"
    )

    func main() {
        var t ITestIntf
        t = NewTest().SetName("张三")
        var s ISubTestIntf
        s = NewSubTest()
        fmt.Println(t.GetName())
        s.SetAge(12).SetSex("女").SetName("李四")//这里不是很完美，父接口的方法必须放到后面。
        fmt.Println("name:",s.GetName(),"sex:", s.GetSex(), "age:", s.GetAge())
    }

