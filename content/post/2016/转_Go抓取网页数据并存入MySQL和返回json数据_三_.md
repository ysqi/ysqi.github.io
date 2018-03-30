
---
date: 2016-12-31T11:34:13+08:00
title: "Go抓取网页数据并存入MySQL和返回json数据<三>"
description: ""
disqus_identifier: 1485833653859308224
slug: "Gozhua-qu-wang-xie-shu-ju-bing-cun-ru-MySQLhe-fan-hui-jsonshu-ju-&amp;lt;san-&amp;gt;"
source: "https://segmentfault.com/a/1190000003810123"
tags: 
- json 
- mysql 
- golang 
topics:
- 编程语言与开发
---

上一节主要实现了使用 [goquery](https://github.com/PuerkitoBio/goquery)
从图片网站
[](http://www.gratisography.com/)<http://www.gratisography.com/>
抓取数据。主要抓取图片的`data-original`、`width`、`height`、`alt`、`type`
五项数据。因此需要先创建数据库和相应的表，在mac上我使用 `Sequel Pro`
数据库管理软件，连接之后创建新的数据库`indiepic`,然后创建表`gratisography`:

    CREATE TABLE `gratisography` (
      `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
      `img_url` varchar(255) DEFAULT NULL,
      `type_name` varchar(50) DEFAULT NULL,
      `title` varchar(255) DEFAULT NULL,
      `width` int(11) DEFAULT NULL,
      `height` int(11) DEFAULT NULL,
      `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=388 DEFAULT CHARSET=utf8;

创建完数据库之后，就开始使用GO来实现连接数据库等操作了。在GO中使用[Go-MySQL-Driver](https://github.com/Go-SQL-Driver/MySQL)
is a lightweight and fast MySQL-Driver for Go's (golang) database/sql
package

文档：[](http://godoc.org/github.com/go-sql-driver/mysql)<http://godoc.org/github.com/go-sql-driver/mysql>

在使用之前需要先使用以下命令获取该包：

    go get github.com/go-sql-driver/mysql

然后在`database.go`中引入：

    package crawldata

    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )

然后写一个打开数据库的方法`OpenDatabase`:

    package crawldata

    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )

    func OpenDatabase() (*sql.DB, error) {
        // 连接数据库
        db, err := sql.Open("mysql", "root:mysql@tcp(xxx.xx.xx.xxx:3306)/databaseName?charset=utf8")
        if err != nil {
            return nil, err
        }
        return db, nil
    }

上一节已在`crawldata.go`中写了`InsertData(&imageDatas)`方法，但是是注释的，先在就需要在该文件中实现该方法。

    package crawldata

    import (
        "database/sql"
        "fmt"
        _ "github.com/go-sql-driver/mysql"
        "strconv"
        s "strings"
    )

    func OpenDatabase() (*sql.DB, error) {
        // 连接数据库
        db, err := sql.Open("mysql", "root:mysql@tcp(xxx.xx.xx.xxx:3306)/databaseName?charset=utf8")
        if err != nil {
            return nil, err
        }
        return db, nil
    }

    /*
       该函数将获取的数据存储到数据库
    */
    func InsertData(datas *ImageDatas) {
        imageDatas := *datas
        // 连接数据库
        db, err := OpenDatabase()
        if err != nil {
            fmt.Printf(s.Join([]string{"连接数据库失败", err.Error()}, "-->"))
        }
        defer db.Close()

        for i := 0; i < len(imageDatas); i++ {
            imageData := imageDatas[i]
            // Prepare statement for inserting data
            imgIns, err := db.Prepare("INSERT INTO gratisography (img_url, type_name, title, width, height) VALUES( ?, ?, ?, ?, ? )") // ? = placeholder
            if err != nil {
                fmt.Println(s.Join([]string{"拼装数据格式", err.Error()}, "-->"))
            }
            defer imgIns.Close() // Close the statement when we leave main()

            img, err := imgIns.Exec(s.Join([]string{"http://www.gratisography.com", imageData.Src}, "/"), imageData.Tp, imageData.Title, imageData.Width, imageData.Height)
            if err != nil {
                fmt.Println(s.Join([]string{"插入数据失败", err.Error()}, "-->"))
            } else {
                success, _ := img.LastInsertId()
                // 数字变成字符串,success是int64型的值，需要转为int，网上说的Itoa64()在strconv包里不存在
                insertId := strconv.Itoa(int(success))
                fmt.Println(s.Join([]string{"成功插入数据：", insertId}, "\t-->\t"))
            }
        }
    }

到此已经完成了数据抓取并存入数据库，在命令行中切换到`$GOPATH/src/indiepic`目录下，然后运行：

    go run indiepic.go

随后就可以看到数据被存入数据库了。\
到这里只实现了数据的获取，但是需要使用GO向外部提供`json`接口，下一节完成数据的获取和使用web框架返回json数据。

