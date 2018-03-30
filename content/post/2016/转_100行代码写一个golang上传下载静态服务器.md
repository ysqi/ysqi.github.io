
---
date: 2016-12-31T11:35:04+08:00
title: "100行代码写一个golang上传下载静态服务器"
description: ""
disqus_identifier: 1485833704755528655
slug: "100hang-dai-ma-xie-yi-ge-golangshang-chuan-xia-zai-jing-tai-fu-wu-qi"
source: "https://segmentfault.com/a/1190000000415336"
tags: 
- golang 
topics:
- 编程语言与开发
---

> 许多朋友开始加入golang的大本营，然后呢都是去看看golang的一些特性，问golang足够简单吗？有什么特性？能做什么？

> 上边那些回答不了，有些学基础的朋友很想做一些东西，然后我就写了这个静态文件服务器，可以上传下载，麻雀虽小，但是包含了很多零碎的小知识点大家一起来巩固一下基础知识吧

        package main

    import (
        "fmt"
        "html/template"
        "io"
        "net/http"
        "os"
        "path/filepath"
        "regexp"
        "strconv"
        "time"
    )

    var mux map[string]func(http.ResponseWriter, *http.Request)

    type Myhandler struct{}
    type home struct {
        Title string
    }

    const (
        Template_Dir = "./view/"
        Upload_Dir   = "./upload/"
    )

    func main() {
        server := http.Server{
            Addr:        ":9090",
            Handler:     &Myhandler{},
            ReadTimeout: 10 * time.Second,
        }
        mux = make(map[string]func(http.ResponseWriter, *http.Request))
        mux["/"] = index
        mux["/upload"] = upload
        mux["/file"] = StaticServer
        server.ListenAndServe()
    }

    func (*Myhandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
        if h, ok := mux[r.URL.String()]; ok {
            h(w, r)
            return
        }
        if ok, _ := regexp.MatchString("/css/", r.URL.String()); ok {
            http.StripPrefix("/css/", http.FileServer(http.Dir("./css/"))).ServeHTTP(w, r)
        } else {
            http.StripPrefix("/", http.FileServer(http.Dir("./upload/"))).ServeHTTP(w, r)
        }

    }

    func upload(w http.ResponseWriter, r *http.Request) {

        if r.Method == "GET" {
            t, _ := template.ParseFiles(Template_Dir + "file.html")
            t.Execute(w, "上传文件")
        } else {
            r.ParseMultipartForm(32 << 20)
            file, handler, err := r.FormFile("uploadfile")
            if err != nil {
                fmt.Fprintf(w, "%v", "上传错误")
                return
            }
            fileext := filepath.Ext(handler.Filename)
            if check(fileext) == false {
                fmt.Fprintf(w, "%v", "不允许的上传类型")
                return
            }
            filename := strconv.FormatInt(time.Now().Unix(), 10) + fileext
            f, _ := os.OpenFile(Upload_Dir+filename, os.O_CREATE|os.O_WRONLY, 0660)
            _, err = io.Copy(f, file)
            if err != nil {
                fmt.Fprintf(w, "%v", "上传失败")
                return
            }
            filedir, _ := filepath.Abs(Upload_Dir + filename)
            fmt.Fprintf(w, "%v", filename+"上传完成,服务器地址:"+filedir)
        }
    }

    func index(w http.ResponseWriter, r *http.Request) {
        title := home{Title: "首页"}
        t, _ := template.ParseFiles(Template_Dir + "index.html")
        t.Execute(w, title)
    }

    func StaticServer(w http.ResponseWriter, r *http.Request) {
        http.StripPrefix("/file", http.FileServer(http.Dir("./upload/"))).ServeHTTP(w, r)
    }

    func check(name string) bool {
        ext := []string{".exe", ".js", ".png"}

        for _, v := range ext {
            if v == name {
                return false
            }
        }
        return true
    }

github地址:[](http://github.com/widuu/staticserver)<http://github.com/widuu/staticserver>

本文转载自我的个人博客[微度网络-网络技术支持中心](http://www.widuu.com/archives/02/959.html)<http://www.widuu.com/archives/02/959.html>

