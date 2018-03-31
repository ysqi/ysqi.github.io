
---
date: 2017-05-24T09:17:28+08:00
title: "Golang读写Excel"
description: ""
disqus_identifier: 1495588648709903671
slug: "Golang-dou-xie--Excel"
source: "https://segmentfault.com/a/1190000009348789"
tags: 
- golang 
- excel 
- openxml 
categories:
- 编程语言与开发
---

[Excelize](https://github.com/Luxurioust/excelize) 是 Golang
编写的一个用来操作 Office Excel 文档类库，基于微软的 Office OpenXML
标准。可以使用它来读取、写入 XLSX 文件。相比较其他的开源类库，Excelize
支持写入原本带有图片(表)的文档，还支持向 Excel
中插入图片，并且在保存后不会丢失图表样式。

### 安装

    go get github.com/Luxurioust/excelize

### 创建 XLSX

    package main

    import (
        "fmt"
        "os"

        "github.com/Luxurioust/excelize"
    )

    func main() {
        xlsx := excelize.CreateFile()
        // Create a new sheet.
        xlsx.NewSheet(2, "Sheet2")
        // Set value of a cell.
        xlsx.SetCellValue("Sheet2", "A2", "Hello world.")
        xlsx.SetCellValue("Sheet1", "B2", 100)
        // Set active sheet of the workbook.
        xlsx.SetActiveSheet(2)
        // Save xlsx file by the given path.
        err := xlsx.WriteTo("./Workbook.xlsx")
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
    }

### 读写已有文档

    package main

    import (
        "fmt"
        "os"
        "strconv"

        "github.com/Luxurioust/excelize"
    )

    func main() {
        xlsx, err := excelize.OpenFile("./Workbook.xlsx")
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        // Get value from cell by given sheet index and axis.
        cell := xlsx.GetCellValue("Sheet1", "B2")
        fmt.Println(cell)
        // Get sheet index.
        index := xlsx.GetSheetIndex("Sheet2")
        // Get all the rows in a sheet.
        rows := xlsx.GetRows("sheet" + strconv.Itoa(index))
        for _, row := range rows {
            for _, colCell := range row {
                fmt.Print(colCell, "\t")
            }
            fmt.Println()
        }
    }

### 向 Excel 中插入图表

    package main

    import (
        "fmt"
        "os"

        "github.com/Luxurioust/excelize"
    )

    func main() {
        categories := map[string]string{"A2": "Small", "A3": "Normal", "A4": "Large", "B1": "Apple", "C1": "Orange", "D1": "Pear"}
        values := map[string]int{"B2": 2, "C2": 3, "D2": 3, "B3": 5, "C3": 2, "D3": 4, "B4": 6, "C4": 7, "D4": 8}
        xlsx := excelize.CreateFile()
        for k, v := range categories {
            xlsx.SetCellValue("Sheet1", k, v)
        }
        for k, v := range values {
            xlsx.SetCellValue("Sheet1", k, v)
        }
        xlsx.AddChart("Sheet1", "E1", `{"type":"bar3D","series":[{"name":"=Sheet1!$A$2","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$2:$D$2"},{"name":"=Sheet1!$A$2","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$3:$D$3"},{"name":"=Sheet1!$A$3","categories":"=Sheet1!$B$1:$D$1","values":"=Sheet1!$B$4:$D$4"}],"title":{"name":"Fruit 3D Line Chart"}}`)
        // Save xlsx file by the given path.
        err := xlsx.WriteTo("./Workbook.xlsx")
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
    }

### 向 Excel 中插入图片

    package main

    import (
        "fmt"
        "os"
        _ "image/gif"
        _ "image/jpeg"
        _ "image/png"

        "github.com/Luxurioust/excelize"
    )

    func main() {
        xlsx, err := excelize.OpenFile("./Workbook.xlsx")
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
        // Insert a picture.
        err = xlsx.AddPicture("Sheet1", "A2", "./image1.gif", "")
        if err != nil {
            fmt.Println(err)
        }
        // Insert a picture to sheet with scaling.
        err = xlsx.AddPicture("Sheet1", "D2", "./image2.jpg", `{"x_scale": 0.5, "y_scale": 0.5}`)
        if err != nil {
            fmt.Println(err)
        }
        // Insert a picture offset in the cell with printing support.
        err = xlsx.AddPicture("Sheet1", "H2", "./image3.gif", `{"x_offset": 15, "y_offset": 10, "print_obj": true, "lock_aspect_ratio": false, "locked": false}`)
        if err != nil {
            fmt.Println(err)
        }
        // Save the xlsx file with the origin path.
        err = xlsx.Save()
        if err != nil {
            fmt.Println(err)
            os.Exit(1)
        }
    }

还有其他一些功能，在这里就不一一列举了，详细使用文档以及获取后期的维护更新可以从项目的主页获取
[github.com/Luxurioust/excelize](https://github.com/Luxurioust/excelize)

