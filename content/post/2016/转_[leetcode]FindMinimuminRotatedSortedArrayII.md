
---
date: 2016-12-31T11:34:39+08:00
title: "[leetcode]FindMinimuminRotatedSortedArrayII"
description: ""
disqus_identifier: 1485833679394627626
slug: "[leetcode]-Find-Minimum-in-Rotated-Sorted-Array-II"
source: "https://segmentfault.com/a/1190000000763035"
tags: 
- golang 
- 算法 
- leetcode 
categories:
- 编程语言与开发
---

code：<https://play.golang.org/p/luj1fdu03F>\
problem:
<https://oj.leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/>

    package main

    import (
        "fmt"
        "math/rand"
        "sort"
        "time"
    )
    //rotate数组方法一
    func rotateArray1(a []int, pos int) {
        if pos < 0 {
            return
        }

        lenA := len(a)
        pos = pos % lenA

        tmpArray := make([]int, pos)
        for i := 0; i < pos; i++ {
            tmpArray[i] = a[i]
        }

        for i := pos; i < lenA; i++ {
            a[i-pos] = a[i]
        }

        for i := 0; i < pos; i++ {
            a[lenA-pos+i] = tmpArray[i]
        }
    }

    //rotate数组方法2
    func rotateArray2(a []int, pos int) {
        if pos < 0 {
            return
        }

        lenA := len(a)
        pos = pos % lenA

        reverseArray(a[:pos])
        reverseArray(a[pos:])
        reverseArray(a)
    }

    //reverse数组，反转
    func reverseArray(a []int) {
        for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
            a[i], a[j] = a[j], a[i]
        }
    }

    func main() {
        len := 20
        a := make([]int, len)
        seedNum := 0
        rand.Seed(int64(seedNum) + time.Now().UnixNano())
        for i := 0; i < len; i++ {
            seedNum++
            a[i] = rand.Intn(20)
        }

        fmt.Println("-------")
        sort.Ints(a)

        rotatePos := rand.Intn(len)
        fmt.Printf("rotate pos:%d\n", rotatePos)
        fmt.Printf("origin array:%v\n", a)
        rotateArray2(a, rotatePos)
        fmt.Printf("rotated array:%v\n", a)
        //sort.Sort(sort.Reverse(sort.IntSlice(a[])))
        fmt.Println(findMin(a))
    }

    func findMin(arrItem []int) int {
        lenItem := len(arrItem)
        low := 0
        high := lenItem - 1
        if arrItem[low] < arrItem[high] {
            return arrItem[low]
        }
        for low < high-1 {
            mid := (low + high) / 2
            if arrItem[low] >= arrItem[high] {
                if arrItem[mid] < arrItem[low] {
                    high = mid
                } else {
                    low = mid
                }

            }
        }
        if arrItem[low] < arrItem[high] {
            return arrItem[low]
        } else {
            return arrItem[high]
        }
    }

