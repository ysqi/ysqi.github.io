
---
date: 2016-12-31T11:34:15+08:00
title: "Go抓取网页数据并存入MySQL和返回json数据&amp;lt;一&amp;gt;"
description: ""
disqus_identifier: 1485833655343593404
slug: "Gozhua-qu-wang-xie-shu-ju-bing-cun-ru-MySQLhe-fan-hui-jsonshu-ju-&amp;lt;yi-&amp;gt;"
source: "https://segmentfault.com/a/1190000003810106"
tags: 
- json 
- mysql 
- golang 
topics:
- 编程语言与开发
---

#### 前言

很久前就想学习`GO`，但是由于准备读研和要实习就一直耽搁没动手，只是偶尔看一下相关的基本语法，并没有将其具体地运用到实际的编码中。大四了，课程一下子少了很多，于是决定用它从网上抓一些图片数据，然后提供接口，为后面学习`iOS`提供一些网络数据。

有关`GO`的介绍我就不在这里说了，对于我这种初学者本来说得就不清不楚，多给自己落下话柄。\
我要实现的功能主要有如下几点：

-   从精美图片网站抓取图片链接等数据；

-   将获取的数据存入`MySQL`数据库；

-   提供一个简单的`json`接口使得自己能通过某链接获取`json`数据。

#### 准备工作

##### 安装GO并配置环境

因为我自己使用的时`OS X`，也写了一个mac安装GO的[文章](http://blog.helloarron.com/2015/08/29/mac-install-go/),如果使用mac的话可以参考一下。windows下百度也会很好解决。

#### 分析小程序

在`$GOPATH/src`下的创建一个项目文件夹`indiepic`作为这次小程序的目录。GO的每一个项目有且仅有一个`package main`，在项目文件夹下新建一个GO文件`indiepic.go`作为主文件：

    package main

    import "fmt"

    func main () {
        fmt.Println("Hello World")
    }

因为后面会启动该文件，然后提供`HTTP`接口提供数据，所以为了可读性将抓取数据并存入数据库等操作放入该项目的一个`包`中，而且抓取数据的操作会很少被操作，不需要在每次启动都执行，所以将其组织到一个`package`中是不错的方法，这样只需在需要抓取的时候在`main`函数中调用接口。

因此，在项目文件夹中新建一个`crawldata`文件夹，该文件就是我们需要的`package`。下面需要的抓取数据和将数据存入数据库以及从数据库中获取数据都写为该包下的一个函数。

在`crawldata`文件夹下新建`crawldata.go`和`database.go`文件。一个与抓取数据有关，一个与数据库存取数据有关。\
文件夹结构如下：

    indiepic
    ├── README.md
    ├── crawldata
    │   ├── crawldata.go
    │   └── database.go
    └── indiepic.go

下一步就开始实现数据抓取部分的功能。\
主要抓取图片网站
[](http://www.gratisography.com/)<http://www.gratisography.com/>

