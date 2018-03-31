
---
date: 2016-12-31T11:34:54+08:00
title: "go语言入门-安装-编写-运行(一)"
description: ""
disqus_identifier: 1485833694876064778
slug: "goyu-yan-ru-men--an-zhuang--bian-xie--yun-hang-(yi-)"
source: "https://segmentfault.com/a/1190000000493069"
tags: 
- golang 
categories:
- 编程语言与开发
---

windows平台\
1.下载go语言包，解压到C:\\go

2.增加了2个全局变量，修改了1个变量\
1、变量名：GOPATH 变量值：e:\\go GO的编译目录在e:\\go这个文件夹里.\
2、变量名：GOROOT 变量值：c:\\go GO的主目录在c:\\go这个文件夹里.\
3、在变量名:PTAH,开始增加C:\\go\\bin;记得一定在结尾加上';'分号.

3.下载并安装Notepad++5.6.8（就不给下载地址了，GOOGLE上一大片）

4.对Notepad++进行配置：\
1、安装好NOTEPAD++后，我们还要给它安装一个插件。安装过程是插件－Plugin
Manage－Show plugin manager（图1）\
2、在Available选单下，找一个叫NppExec的插件，选中后，点击Install安装。见下图。\
\
3、 插件安装好后，我们要对该插件进行一下设定。见下图\
\
4、具体设定如下\

5.测试\
打开Notepad++，点格式－以UTF-8无BOM格式编码，我们要以UTF－8格式写一段GO程序。这里也要注意一下，写程序前，一定要用无BOM的UTF-8模式写，也就是先变模式后写程序，不然Notepad++会以默认的ANSI格式编码，其结果是英文可见，而中文则成了乱码。下面有写好的程序，大家可以直接粘贴，（见下图）：\

把以下程序粘贴到Notepad++里:

    package main   
    import "fmt"  
    func main() {   

      fmt.Printf("你好，中国！ Hello, World!\n")   

    }  

    保存到e:\go（GOPATH 目录是存放本地的go文件里的）

6.运行\
cd e:\\go 我们先进入e盘的GO目录中\
go run hello.go

这样就完成了第一个go语言的程序，hello word!

