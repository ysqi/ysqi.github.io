
---
date: 2016-12-31T11:34:26+08:00
title: "go语言的一些吐槽"
description: ""
disqus_identifier: 1485833666714370772
slug: "goyu-yan-de-yi-xie-tu-cao"
source: "https://segmentfault.com/a/1190000002981834"
tags: 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

struct的方法,如果receiver非指针,则调用这个方法无法改变对象状态,因为传递给方法的对象是原对象的一个拷贝,所有的改变都被作用在这个拷贝上而非原对象上.

    type st struct{
        val uint32
    }

    func (this st) Show(){
        fmt.Printf("Show:%d\n",this.val)
    }

    func (this st) Increase(){
        this.val += 1
        fmt.Printf("Increase:%d\n",this.val)
    }

    func main(){
        b := st{val:10}
        b.Increase()
        b.Show()
    }

输出:

    Increase:11
    Show:10

对于习惯了C++的程序员,总会认为调用一个对象的非const方法是可以改变那个对象的内部状态.但是这种思维如果延续到go中将会导致很难查找的bug.

到底是对象实现了接口还是指向对象的指针实现了接口

先来看以下代码:

    package main

    import "fmt"

    type intf interface{
        Show()
    }

    type st struct{
        val uint32
    }

    func (this *st) Show(){
        this.val += 1
        fmt.Printf("%d\n",this.val)
    }

    func main(){
        var a intf
        b := st{val:10}
        a = b
        a.Show()
    }

直观上我们认为st实现了intf接口,所以可以用b对a赋值,而实际上运行这段代码将会报错:

    # command-line-arguments
    test/test.go:21: cannot use b (type st) as type intf in assignment:
            st does not implement intf (Show method has pointer receiver)

这段提示说st没有实现intf接口,因为Show方法的receiver是一个指针.对代码稍作修改:

    func main(){
        var a intf
        b := &st{val:10}
        a = b
        a.Show()
    }

这次代码可以正确运行了,此时b应该是一个指向st的指针.这是说\*st实现了intf接口?

现在我再把Show的定义改一下,main不变:

    func (this st) Show(){
        this.val += 1
        fmt.Printf("%d\n",this.val)
    }

代码还是可以正确运行,同时如果把main恢复成下面这样:

    func main(){
        var a intf
        b := st{val:10}
        a = b
        a.Show()
    }

代码也是正确的.

也就是说,实现接口的方法时receiver是指针那么只能用实现类型的指针对接口赋值,否则既能用实现类型的值也能用实现类型的指针对接口赋值.

最后再来看个例子,在一段网络代码中,我想将net.Conn转换成net.TCPConn,如下代码报错:

    tcpconn := this.Conn.(net.TCPConn)

    # kendynet-go/socket
    ./tcpsession.go:156: impossible type assertion:
            net.TCPConn does not implement net.Conn (Close method has pointer receiver)
    Makefile:2: recipe for target 'all' failed

正确的做法是:

    tcpconn := this.Conn.(*net.TCPConn)

这是否证明了,是\*net.TCPConn而不是net.TCPConn实现了net.Conn接口,

