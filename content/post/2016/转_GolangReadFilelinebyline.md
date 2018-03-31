
---
date: 2016-12-31T11:34:42+08:00
title: "GolangReadFilelinebyline"
description: ""
disqus_identifier: 1485833682071839928
slug: "Golang-Read-File-line-by-line"
source: "https://segmentfault.com/a/1190000000700795"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

学习什么语言都得从读文件开始，好像记得一个大神说过计算机编程就是"打开文件，操作，关闭文件"。初学Golang就记一下go语言的文件操作

**Read File**

`func main(){         rw,err := os.Open("")         if err != nil {             panic(err)         }         defer rw.Close()         rb := bufio.NewReader(rw)         for {             line, _, err := rb.ReadLine()             if err == io.EOF {                 break             }             //do something             fmt.Println(string(line))         } }`\
`func main(){         rw,err := os.Open("")         if err != nil {             panic(err)         }         defer rw.Close()         sb := bufio.NewScanner(rw)         for sb.Scan() {             //do something             fmt.Println(sb.Text())         }         if err := sb.Err(); err !=nil {             panic(err)         } }`\
**Write File**\
`func main(){         fw,err := os.OpenFile("",os.O_WRONLY|os.O_CREATE|os.O_APPEND,0644)         if err != nil {             panic(err)         }         defer fw.Close()         wb := bufio.NewWriter(fw)         wb.WriteString("hello world\n")         wb.Flush() }`\
**Read Dir**\
`func main(){         fw,err := os.OpenFile("",os.O_WRONLY|os.O_CREATE|os.O_APPEND,0644)         if err != nil {             panic(err)         }         defer fw.Close()         fileinfos, err := fw.Readdir(0)         if err != nil {             panic(err)         }         for _, fileinfo := range fileinfos {             //do something             fmt.Println(fileinfo.Name(), fileinfo.Size())         } }`

