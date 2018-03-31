
---
date: 2016-12-31T11:34:43+08:00
title: "golang实现快速排序"
description: ""
disqus_identifier: 1485833683098792792
slug: "golangshi-xian-kuai-su-pai-xu"
source: "https://segmentfault.com/a/1190000000665655"
tags: 
- 算法 
- golang 
categories:
- 编程语言与开发
---

快速排序的原理就不介绍了。在网上看到一个有趣的视频，大家可以看看，非常详细且有趣。\
[快速排序视频](http://www.tudou.com/v/gtnrNh7yh6I/&resourceId=0_04_05_99/v.swf)\
代码：<https://play.golang.org/p/Fw5gtzrPj0>

    package main

    import (
        "fmt"
    )

    func main() {
        var sortArray = []int{3, 41, 24, 76, 11, 45, 3, 3, 64, 21, 69, 19, 36}
        fmt.Println(sortArray)
        qsort(sortArray, 0, len(sortArray)-1)
        fmt.Println(sortArray)
    }

    func qsort(array []int, low, high int) {
        if low < high {
            m := partition(array, low, high)
            // fmt.Println(m)
            qsort(array, low, m-1)
            qsort(array, m+1, high)
        }
    }

    func partition(array []int, low, high int) int {
        key := array[low]
        tmpLow := low
        tmpHigh := high
        for {
            //查找小于等于key的元素，该元素的位置一定是tmpLow到high之间，因为array[tmpLow]及左边元素小于等于key，不会越界
            for array[tmpHigh] > key {
                tmpHigh--
            }
            //找到大于key的元素，该元素的位置一定是low到tmpHigh+1之间。因为array[tmpHigh+1]必定大于key
            for array[tmpLow] <= key && tmpLow < tmpHigh {
                tmpLow++
            }

            if tmpLow >= tmpHigh {
                break
            }
            // swap(array[tmpLow], array[tmpHigh])
            array[tmpLow], array[tmpHigh] = array[tmpHigh], array[tmpLow]
            fmt.Println(array)
        }
        array[tmpLow], array[low] = array[low], array[tmpLow]
        return tmpLow
    }

