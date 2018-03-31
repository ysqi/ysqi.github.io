
---
date: 2016-12-31T11:34:38+08:00
title: "[leetcode]wordsearch"
description: ""
disqus_identifier: 1485833678189887294
slug: "[leetcode]-wordsearch"
source: "https://segmentfault.com/a/1190000002391006"
tags: 
- golang 
- 算法 
- leetcode 
categories:
- 编程语言与开发
---

problem: <https://oj.leetcode.com/problems/word-search/>\
代码：<https://play.golang.org/p/d5wP691Pbg>

    package main

    import (
        "fmt"
    )

    func main() {
        arr := [][]byte{
            {'A', 'B', 'C', 'E'},
            {'S', 'F', 'C', 'S'},
            {'A', 'D', 'E', 'E'},
        }
        //word := []byte("ABCESC")
        word := []byte("SFCE")
        arrI, arrJ, ok := FindFirstCharPositions(arr, word[0])
        if ok {
            fmt.Printf("%d, %d\n", arrI, arrJ)
        }
        fmt.Println(DFSTraverse(arr, word, arrI, arrJ))
    }

    func DFSTraverse(arr [][]byte, word []byte, arrI []int, arrJ []int) (ret bool) {
        //初始化visited函数
        lenArr := len(arr)
        visited := make([][]bool, lenArr)
        for i := 0; i < lenArr; i++ {
            visited[i] = make([]bool, len(arr[i]))
        }
        lenArrI := len(arrI)
        //lenArrJ := len(arrJ)
        for i := 0; i < lenArrI; i++ {
            //fmt.Println(i)
            ret = DFS(arr, word, 0, visited, arrI[i], arrJ[i])
            if ret == true {
                return true
            }
        }
        return
    }

    //dfs traverse, when found return true
    func DFS(arr [][]byte, word []byte, index int, visited [][]bool, i int, j int) (ret bool) {
        if i < 0 || j < 0 || i >= len(visited) || j >= len(visited[i]) || visited[i][j] {
            return false
        }
        if index == len(word) {
            return true
        }
        if arr[i][j] != word[index] {
            return false
        }

        fmt.Printf("%c:%d,%d ", word[index], i, j)
        visited[i][j] = true //标识为已访问
        index++
        //fmt.Println("yes")
        ret = DFS(arr, word, index, visited, i-1, j) ||
            DFS(arr, word, index, visited, i, j+1) ||
            DFS(arr, word, index, visited, i+1, j) ||
            DFS(arr, word, index, visited, i, j-1)
        visited[i][j] = false
        return ret
    }

    func FindFirstCharPositions(arr [][]byte, c byte) (arrI []int, arrJ []int, ok bool) {
        lenArrRow := len(arr)
        if lenArrRow == 0 {
            return
        }
        lenArrCol := len(arr[0])

        for i := 0; i < lenArrRow; i++ {
            for j := 0; j < lenArrCol; j++ {
                if arr[i][j] == c {
                    arrI = append(arrI, i)
                    arrJ = append(arrJ, j)
                    ok = true
                }
            }
        }
        return
    }


