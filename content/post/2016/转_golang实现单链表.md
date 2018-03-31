
---
date: 2016-12-31T11:32:53+08:00
title: "golang实现单链表"
description: ""
disqus_identifier: 1485833573941499491
slug: "golang-shi-xian-chan-lian-biao"
source: "https://segmentfault.com/a/1190000007442806"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

    package main

    //链表实现
    import (
        "fmt"
        "os"
    )

    //定义错误常量
    const (
        ERROR = -1000000001
    )

    //定义元素类型
    type Element int64

    //定义节点
    type LinkNode struct {
        Data Element   //数据域
        Nest *LinkNode //指针域，指向下一个节点
    }

    //函数接口
    type LinkNoder interface {
        Add(head *LinkNode, new *LinkNode)              //后面添加
        Delete(head *LinkNode, index int)               //删除指定index位置元素
        Insert(head *LinkNode, index int, data Element) //在指定index位置插入元素
        GetLength(head *LinkNode) int                   //获取长度
        Search(head *LinkNode, data Element)            //查询元素的位置
        GetData(head *LinkNode, index int) Element      //获取指定index位置的元素
    }

    //添加 头结点，数据
    func Add(head *LinkNode, data Element) {
        point := head //临时指针
        for point.Nest != nil {
            point = point.Nest //移位
        }
        var node LinkNode  //新节点
        point.Nest = &node //赋值
        node.Data = data

        head.Data = Element(GetLength(head)) //打印全部的数据

        if GetLength(head) > 1 {
            Traverse(head)
        }

    }

    //删除 头结点 index 位置
    func Delete(head *LinkNode, index int) Element {
        //判断index合法性
        if index < 0 || index > GetLength(head) {
            fmt.Println("please check index")
            return ERROR
        } else {
            point := head
            for i := 0; i < index-1; i++ {
                point = point.Nest //移位
            }
            point.Nest = point.Nest.Nest //赋值
            data := point.Nest.Data
            return data
        }
    }

    //插入 头结点 index位置 data元素
    func Insert(head *LinkNode, index int, data Element) {
        //检验index合法性
        if index < 0 || index > GetLength(head) {
            fmt.Println("please check index")
        } else {
            point := head
            for i := 0; i < index-1; i++ {
                point = point.Nest //移位
            }
            var node LinkNode //新节点，赋值
            node.Data = data
            node.Nest = point.Nest
            point.Nest = &node
        }
    }

    //获取长度 头结点
    func GetLength(head *LinkNode) int {
        point := head
        var length int
        for point.Nest != nil {
            length++
            point = point.Nest
        }
        return length
    }

    //搜索 头结点 data元素
    func Search(head *LinkNode, data Element) {
        point := head
        index := 0
        for point.Nest != nil {
            if point.Data == data {
                fmt.Println(data, "exist at", index, "th")
                break
            } else {
                index++
                point = point.Nest
                if index > GetLength(head)-1 {
                    fmt.Println(data, "not exist at")
                    break
                }
                continue
            }
        }
    }

    //获取data 头结点 index位置
    func GetData(head *LinkNode, index int) Element {
        point := head
        if index < 0 || index > GetLength(head) {
            fmt.Println("please check index")
            return ERROR
        } else {
            for i := 0; i < index; i++ {
                point = point.Nest
            }
            return point.Data
        }
    }

    //遍历 头结点
    func Traverse(head *LinkNode) {
        point := head.Nest
        for point.Nest != nil {
            fmt.Println(point.Data)
            point = point.Nest
        }
        fmt.Println("Traverse OK!")
    }

    //主函数测试
    func main() {
        var head LinkNode = LinkNode{Data: 0, Nest: nil}
        head.Data = 0
        var nodeArray []Element
        for i := 0; i < 10; i++ {
            nodeArray = append(nodeArray, Element(i+1+i*100))
            Add(&head, nodeArray[i])

        }

        Delete(&head, 3)
        Search(&head, 2032)
        Insert(&head, 23, 10010)
        Traverse(&head)
        fmt.Println("data is", GetData(&head, 6))
        fmt.Println("length:", GetLength(&head))
        os.Exit(0)
    }

