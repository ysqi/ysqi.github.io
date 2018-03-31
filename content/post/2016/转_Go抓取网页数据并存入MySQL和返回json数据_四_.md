
---
date: 2016-12-31T11:34:13+08:00
title: "Go抓取网页数据并存入MySQL和返回json数据<四>"
description: ""
disqus_identifier: 1485833653402386329
slug: "Gozhua-qu-wang-xie-shu-ju-bing-cun-ru-MySQLhe-fan-hui-jsonshu-ju-&amp;lt;si-&amp;gt;"
source: "https://segmentfault.com/a/1190000003810156"
tags: 
- json 
- mysql 
- golang 
categories:
- 编程语言与开发
---

上一节已将将需要的数据从网站[](http://www.gratisography.com/)<http://www.gratisography.com/>
抓取并存入数据库【使用`crawldata.go`中的`InsertData(&imageDatas)`函数】，现在需要将数据从数据库`indiepic`的表`gratisography`中取出并然会`json`格式的数据。

项目文件夹结构如下：

    indiepic
    ├── README.md
    ├── crawldata
    │   ├── crawldata.go
    │   └── database.go
    └── indiepic.go

现在将获取数据的函数写在`database.go`中：

    func GetAllImages() (imageDatas ImageDatas, err error) {
        // 连接数据库
        db, err := OpenDatabase()
        if err != nil {
            fmt.Printf(s.Join([]string{"连接数据库失败", err.Error()}, "-->"))
            return nil, err
        }
        defer db.Close()

        // Prepare statement for inserting data
        imgOut, err := db.Query("SELECT * FROM gratisography")
        if err != nil {
            fmt.Println(s.Join([]string{"获取数据失败", err.Error()}, "-->"))
            return nil, err
        }
        defer imgOut.Close()

        // 定义扫描select到的数据库字段的变量
        var (
            id          int
            img_url     string
            type_name   string
            title       string
            width       int
            height      int
            create_time string
        )
        for imgOut.Next() {
            // db.Query()中select几个字段就需要Scan多少个字段
            err := imgOut.Scan(&id, &img_url, &type_name, &title, &width, &height, &create_time)
            if err != nil {
                fmt.Println(s.Join([]string{"查询数据失败", err.Error()}, "-->"))
                return nil, err
            } else {
                imageData := ImageData{img_url, type_name, title, width, height}
                imageDatas = append(imageDatas, imageData)
            }
        }

        return imageDatas, nil
    }

值得一提的是在`SELECT`语句中拿到多少个字段就需要在`Scan`的时候使用变量去获取多少个字段，否测会报错。所以建议不要像上面那样`SELECT *`，而是指定需要的字段如`SELECT id,img_url,title FROM tableName`。\
`GetAllImages()`函数返回两个参数`imageDatas ImageDatas`、`err error`。

虽然使用GO自己写一个HTTP请求很简单，但为了更好地处理路由和数据，这里使用一个web框架[martini](https://github.com/go-martini/martini)
Classy web framework for Go。\
在使用之前需要先获取：

    go get github.com/go-martini/martini

在`indiepic.go`中引入`martini`:

    import (
        "github.com/go-martini/martini"
    )

定义一个结构体`Results`用于表示输出结果的数据结构：

    type Results struct {
        Err   int                  // 错误码
        Msg   string               // 错误信息
        Datas crawldata.ImageDatas // 数据，无数据时为nil
    }

因为需要输出`json`格式数据，所以需要用`martini`的`encoder`中间件资源,使用下面命令获取：

    go get github.com/martini-contrib/encoder

然后`import`:

    import (
        "github.com/martini-contrib/encoder"
    )

因为数据已经抓取完存入数据库了，所以在`main`函数中，就不在需要调用`crawldata.Crawl()`了，将其注释掉。然后编写如下代码：

    func main() {
        // 使用crawldata包里面的Crawl()抓取需要的数据存到数据库
        // crawldata.Crawl()

        m := martini.New()
        route := martini.NewRouter()

        var (
            results Results
            err     error
        )

        m.Use(func(c martini.Context, w http.ResponseWriter, r *http.Request) {
            // 将encoder.JsonEncoder{}按照encoder.Encoder接口（注意大小写）类型注入到内部
            c.MapTo(encoder.JsonEncoder{}, (*encoder.Encoder)(nil))
            w.Header().Set("Content-Type", "application/json; charset=utf-8")
        })

        route.Get("/", func(enc encoder.Encoder) (int, []byte) {
            result := Results{10001, "Not Found Data", nil}
            return http.StatusOK, encoder.Must(enc.Encode(result))
        })

        route.Get("/api", func(enc encoder.Encoder) (int, []byte) {
            results.Datas, err = crawldata.GetAllImages()
            if err != nil {
                fmt.Println(s.Join([]string{"获取数据失败", err.Error()}, "-->"))
                result := Results{10001, "Data Error", nil}
                return http.StatusOK, encoder.Must(enc.Encode(result))
            } else {
                results.Err = 10001
                results.Msg = "获取数据成功"
                return http.StatusOK, encoder.Must(enc.Encode(results))
            }
        })

        m.Action(route.Handle)
        m.Run()
    }

`import`部分:

    import (
        "fmt"
        "github.com/go-martini/martini"
        "github.com/martini-contrib/encoder"
        "indiepic/crawldata"
        "net/http"
        s "strings"
    )

然后运行：

    go run indiepic.go

在浏览器中访问 <http://127.0.0.1>:3000/api
即可看到输出的`json`数据。如果安装了`postman`可以使用`postman`访问地址，查看时选择`json`。

到这里这个例子就结束了。源码见[GitHub](https://github.com/ArronYR/GO_CrawlData_MySQL)。

