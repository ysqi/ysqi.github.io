
---
date: 2016-12-31T11:34:14+08:00
title: "Go抓取网页数据并存入MySQL和返回json数据<二>"
description: ""
disqus_identifier: 1485833654631160567
slug: "Gozhua-qu-wang-xie-shu-ju-bing-cun-ru-MySQLhe-fan-hui-jsonshu-ju-&amp;lt;er-&amp;gt;"
source: "https://segmentfault.com/a/1190000003810115"
tags: 
- json 
- mysql 
- golang 
topics:
- 编程语言与开发
---

上一节已经说明了要做什么，以及整个小程序的目录结构，接下来就开始编码部分。\
首先在入口文件中引入项目下的包`crawldata`,然后调用其中抓取数据的函数，暂时取名为`Crawl`:

    package main

    import (
        "fmt"
        "indiepic/crawldata"
    )

    func main () {
        // 使用crawldata包里面的Crawl()抓取需要的数据存到数据库
        crawldata.Crawl()
        fmt.Println("主函数")
    }

然后就是实现包`crawldata`里面的`Crawl`函数。将该函数放在`crawldata.go`文件中：

    package crawldata

    import (
        "fmt"
    )

    func Crawl() {
        fmt.Println("包crawldata中的Crawl函数")
    }

查看网站
[](http://www.gratisography.com/)<http://www.gratisography.com/>，然后审查元素找到某张图片，在图片主要包含了`src`、`data-original`、`width`、`height`、`alt`等信息，首先要明确一点的是这个网站使用了图片的`lazy`加载（在每个`li`标签上可以看出来），所以真正的图片`URL`是`data-original`指定的值而不是`src`，`src`值会在图片加载完成之后被赋为`data-original`的值。另外在网站上有一个分类，所以需存储一下每一张图片的分类，在抓取的时候也是直接通过分类去抓取。\
因此我们需要定义一个结构体来表示每一条数据包含的数据,以及用于存储全部数据的一个切片，然后在`Crawl`函数中使用。如下：

    package crawldata

    import (
        "fmt"
        "github.com/PuerkitoBio/goquery"
        "strconv"
        s "strings"
    )

    // 定义一个存储一条数据的结构体
    type ImageData struct {
        Src    string
        Tp     string
        Title  string
        Width  int
        Height int
    }

    // 定义切片用于存储抓取的全部数据
    type ImageDatas []ImageData

    func Crawl() {
        fmt.Println("包crawldata中的Crawl函数")

        // 定义一个切片存储所有数据
        var datas ImageDatas
        // 抓取数据
        imageDatas := CrawlData(&datas)
    }

在上面的`Crawl`函数中又调用了一个`CrawlData`函数，该函数接受一个用于存储数据的`datas`变量，并返回抓取后的所有数据。

    /*
       该函数用来抓取数据，并将存储的值返回到主函数
    */
    func CrawlData(datas *ImageDatas) (imageDatas ImageDatas) {
        imageDatas = *datas
        // 规定抓取时匹配的元素
        var types = [...]string{
            "people",
            "objects",
            "whimsical",
            "nature",
            "urban",
            "animals"}

        doc, err := goquery.NewDocument("http://www.gratisography.com/")
        if err != nil {
            fmt.Printf(err.Error())
        }

        for _, tp := range types {
            doc.Find("#container ul").Find(s.Join([]string{".", tp}, "")).Each(func(i int, s *goquery.Selection) {
                img := s.Find("img.lazy")
                src, _ := img.Attr("data-original")
                title, _ := img.Attr("alt")
                width, _ := img.Attr("width")
                height, _ := img.Attr("height")

                // 将宽度和高度的字符串类型转为数值型
                wd, error := strconv.Atoi(width)
                if error != nil {
                    fmt.Println("字符串转换成整数失败")
                }
                hg, error := strconv.Atoi(height)
                if error != nil {
                    fmt.Println("字符串转换成整数失败")
                }
                // fmt.Printf("Review %d: %s - %s - %s - %d - %d\n", i, src, tp, title, wd, hg)
                imageData := ImageData{src, tp, title, wd, hg}
                imageDatas = append(imageDatas, imageData)
            })
        }
        return
    }

定义了一个数组`types`用于存放分类，然后根据分类去获得各分类下的图片。

数据抓取使用
[goquery](https://github.com/PuerkitoBio/goquery),在运行之前需要使用如下命令获取`goquery`

    go get github.com/PuerkitoBio/goquery

`goquery`的用法和`jQuery`很类似，直接通过网页中的一些标签、class、id等获取，更多的使用方法可以查看文档。

这里还使用了`strconv`中的函数转换类型，使用`strings`处理字符串。这两个都是GO的内置包，不需要其他运行命令获取。

在`Crawl`函数中可以使用`for`输出看获取到的数据，如：

    func Crawl() {
        // 定义一个切片存储所有数据
        var datas ImageDatas
        // 抓取数据
        imageDatas := CrawlData(&datas)
        for i := 0; i < len(imageDatas); i++ {
            fmt.Println(imageDatas[i].Src, imageDatas[i].Title, imageDatas[i].Tp, imageDatas[i].Height, imageDatas[i].Width)
        }
        // 或者
        for _, imageData := range imageDatas {
            fmt.Println(imageData.Src, imageData.Title, imageData.Tp, imageData.Height, imageData.Width)
        }

        // 将数据插入数据库
        // InsertData(&imageDatas)
    }

下一步实现将获取的数据插入数据库的方法`InsertData(&imageDatas)`。

