
---
date: 2016-12-31T11:32:32+08:00
title: "golang对自定义类型排序"
description: ""
disqus_identifier: 1485833552640028095
slug: "golang-dui-zi-ding-yi-lei-xing-pai-xu"
source: "https://segmentfault.com/a/1190000008062661"
tags: 
- golang 
- sort 
- struct 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

golang 对自定义类型排序
=======================

> 在实际项目中，我们常常需要根据一个结构体类型的某个字段进行排序。之前遇到这个问题不知道如何解决，后来在网上搜索了相关问题，找到了一些好的解决方案，此处参考下，做个总结吧。

由于 golang 的 sort 包本身就提供了相应的功能，
我们就没必要重复的造个轮子了，来看看如何利用 sort 包来实现吧。

sort包浅谈
----------

> sort 包
> 在**内部实现**了四种基本的排序算法：插入排序（insertionSort）、归并排序（symMerge）、堆排序（heapSort）和快速排序（quickSort）；
> sort
> 包会依据实际数据自动选择最优的排序算法。所以我们写代码时只需要考虑实现
> sort.Interface 这个类型就可以了。

### 粗略的看看sort包

    func Sort(data Interface) {
        // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
        n := data.Len()
        maxDepth := 0
        for i := n; i > 0; i >>= 1 {
            maxDepth++
        }
        maxDepth *= 2
        quickSort(data, 0, n, maxDepth)
    }

    type Interface interface {
        // Len is the number of elements in the collection.
        Len() int
        // Less reports whether the element with
        // index i should sort before the element with index j.
        Less(i, j int) bool
        // Swap swaps the elements with indexes i and j.
        Swap(i, j int)
    }
    // 内部实现的四种排序算法
    // 插入排序
    func insertionSort(data Interface, a, b int)
    // 堆排序
    func heapSort(data Interface, a, b int)
    // 快速排序
    func quickSort(data Interface, a, b, maxDepth int)
    // 归并排序
    func symMerge(data Interface, a, m, b int)

所以要调用 sort.Sort() 来实现自定义类型排序，只需要我们的类型实现
Interface 接口类型中的三个方法即可。

先看看 sort 包本身对于 \[\]int 类型如何排序

    // 首先定义了一个[]int类型的别名IntSlice 
    type IntSlice []int
    // 获取此 slice 的长度
    func (p IntSlice) Len() int           { return len(p) }
    // 比较两个元素大小 升序
    func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
    // 交换数据
    func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
    // sort.Ints()内部调用Sort() 方法实现排序
    // 注意 要先将[]int 转换为 IntSlice类型 因为此类型才实现了Interface的三个方法 
    func Ints(a []int) { Sort(IntSlice(a)) }

照葫芦画瓢 我们来对自定义的结构体类型进行降序排序

    package main

    import (
        "fmt"
        "sort"
    )

    type Person struct {
        Name string
        Age  int
    }

    type Persons []Person
    // 获取此 slice 的长度
    func (p Persons) Len() int { return len(p) }
    // 根据元素的年龄降序排序 （此处按照自己的业务逻辑写） 
    func (p Persons) Less(i, j int) bool {
        return p[i].Age > p[j].Age
    }
    // 交换数据
    func (p Persons) Swap(i, j int) { p[i], p[j] = p[j], p[i] }
    func main() {
        persons := Persons{
            {
                Name: "test1",
                Age:  20,
            },
            {
                Name: "test2",
                Age:  22,
            },
            {
                Name: "test3",
                Age:  21,
            },
        }

        fmt.Println("排序前")
        for _, person := range persons {
            fmt.Println(person.Name, ":", person.Age)
        }
        sort.Sort(persons)
        fmt.Println("排序后")
        for _, person := range persons {
            fmt.Println(person.Name, ":", person.Age)
        }
    }

其实，一般 Len() 和 Swap() 基本不做改变，只有涉及到元素比较的 Less()
方法会有所改变。\
当我们对某一个结构体中多个字段进行排序时怎么办，难道每排序一个就写下这三个方法么，当然不是。我们可以利用嵌套结构体来解决这个问题。**因为嵌套结构体可以继承父结构体的所有属性和方法**\
比如我想对上面 Person 的 Name 字段和 Age
对要排序，我们可以利用嵌套结构体来改进一下。

    package main

    import (
        "fmt"
        "sort"
    )

    type Person struct {
        Name string
        Age  int
    }

    type Persons []Person

    // Len()方法和Swap()方法不用变化
    // 获取此 slice 的长度
    func (p Persons) Len() int { return len(p) }

    // 交换数据
    func (p Persons) Swap(i, j int) { p[i], p[j] = p[j], p[i] }

    // 嵌套结构体  将继承 Person 的所有属性和方法
    // 所以相当于SortByName 也实现了 Len() 和 Swap() 方法
    type SortByName struct{ Persons }

    // 根据元素的姓名长度降序排序 （此处按照自己的业务逻辑写）
    func (p SortByName) Less(i, j int) bool {
        return len(p.Persons[i].Name) > len(p.Persons[j].Name)
    }

    type SortByAge struct{ Persons }

    // 根据元素的年龄降序排序 （此处按照自己的业务逻辑写）
    func (p SortByAge) Less(i, j int) bool {
        return p.Persons[i].Age > p.Persons[j].Age
    }

    func main() {
        persons := Persons{
            {
                Name: "test123",
                Age:  20,
            },
            {
                Name: "test1",
                Age:  22,
            },
            {
                Name: "test12",
                Age:  21,
            },
        }

        fmt.Println("排序前")
        for _, person := range persons {
            fmt.Println(person.Name, ":", person.Age)
        }
        sort.Sort(SortByName{persons})
        fmt.Println("排序后")
        for _, person := range persons {
            fmt.Println(person.Name, ":", person.Age)
        }
    }

