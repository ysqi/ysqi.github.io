
---
date: 2016-12-31T11:34:37+08:00
title: "[leetcode]wordladderii"
description: ""
disqus_identifier: 1485833677982663960
slug: "[leetcode]-wordladder-ii"
source: "https://segmentfault.com/a/1190000002403983"
tags: 
- golang 
- 算法 
- leetcode 
topics:
- 编程语言与开发
---

problem: <https://oj.leetcode.com/problems/word-ladder-ii/>\
代码：<https://play.golang.org/p/qdmadQUcEC>

    package main

    import (
        "fmt"
    )

    func main() {
        start := "hit"
        end := "cog"
        dict := []string{"hot", "dot", "dog", "lot", "log"}
        dict = append(dict, start, end)

        mapAdjacency := createAdjacencyGraph(dict, start, end)
        findLadders(mapAdjacency, start, end)
    }

    func findLadders(mapAdjacency map[string][]string, start, end string) {
        visited := make(map[string]int, len(mapAdjacency))
        visited[start] = 1

        queue := make([]string, len(mapAdjacency))
        queue = append(queue, start)

        mapParent := make(map[string][]string, len(mapAdjacency))

        bfs(mapAdjacency, visited, queue, mapParent, start, end)

        stack := []string{}
        stack = append(stack, end)
        printShortestPath(mapParent, stack, start, end)
    }

    func printShortestPath(mapParent map[string][]string, stack []string, start, end string) {
        if end == start {
            printStack(stack)
            return
        }

        for _, v := range mapParent[end] {
            stack = append(stack, v)
            printShortestPath(mapParent, stack, start, v)
            stack = stack[:len(stack)-1]
        }
    }

    func printStack(stack []string) {
        lenStack := len(stack)
        for i := lenStack - 1; i >= 0; i-- {
            fmt.Printf("%s ", stack[i])
        }
        fmt.Println()
    }

    func bfs(mapAdjacency map[string][]string, visited map[string]int, queue []string,
        mapParent map[string][]string, start string, end string) {
        var cur string
        for len(queue) > 0 {
            //出队列
            cur, queue = queue[0], queue[1:]
            for _, value := range mapAdjacency[cur] {
                if visited[value] == 0 {
                    //入队列
                    queue = append(queue, value)
                    visited[value] = visited[cur] + 1
                    mapParent[value] = append(mapParent[value], cur)
                } else if visited[value] > visited[cur] {
                    mapParent[value] = append(mapParent[value], cur)
                }

            }
        }

    }

    func createAdjacencyGraph(dict []string, start, end string) (mapAdjacency map[string][]string) {
        mapAdjacency = make(map[string][]string)
        lenWord := len(start)
        for _, vi := range dict {
            for _, vj := range dict {
                count := 0
                if vi != vj {
                    for k := 0; k < lenWord; k++ {
                        if vi[k] == vj[k] {
                            count++
                        }
                    }
                    if count == lenWord-1 { //find adjacency
                        mapAdjacency[vi] = append(mapAdjacency[vi], vj)
                    }
                }

            }
        }

        return
    }


