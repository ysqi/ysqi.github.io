
---
date: 2016-12-31T11:33:00+08:00
title: "使用go的io_Pipe优雅的优化中间缓存"
description: ""
disqus_identifier: 1485833580697344854
slug: "shi-yong-gode-io_Pipeyou-ya-de-you-hua-zhong-jian-huan-cun"
source: "https://segmentfault.com/a/1190000007172011"
tags: 
- performance 
- pipe 
- io 
- golang 
topics:
- 编程语言与开发
---

### BEFORE

今天发现，go的优势除了它的轻量线程(goroutine)提供了更方便灵活的并发编程模式之外，它的I/O机制也设计的非常给力。

之前，我在向其他服务器发送json数据时，都需要先声明一个`bytes`缓存，然后通过`json`库把结构体中的内容mashal成字节流，再通过Post函数发送。\
代码如下：

    package main

    import (
            "bytes"
            "encoding/json"
            "io/ioutil"
            "log"
            "net/http"
    )

    func init() {
            log.SetFlags(log.Lshortfile)
    }

    func main() {
            cli := http.Client{}

            msg := struct {
                Name, Addr string
                Price      float64
            }{
                Name:  "hello",
                Addr:  "beijing",
                Price: 123.56,
            }
            buf := bytes.NewBuffer(nil)
            json.NewEncoder(buf).Encode(msg)
            resp, err := cli.Post("http://localhost:9999/json", "application/json", buf)
            
            if err != nil {
                log.Fatalln(err)
            }

            body := resp.Body
            defer body.Close()

            if body_bytes, err := ioutil.ReadAll(body); err == nil {
                log.Println("response:", string(body_bytes))
            } else {
                log.Fatalln(err)
            }
    }

这样就总是需要我们在内存中预先分配一个缓存，大部分情况下其实还好，**BUT**
如果需要发送的数据很大，就会严重影响性能了。

### AFTER

go设计了一个Pipe函数帮我们解决这个问题，于是我的代码可以改为：

    package main

    import (
            "encoding/json"
            "io"
            "io/ioutil"
            "log"
            "net/http"
            "time"
    )

    func init() {
            log.SetFlags(log.Lshortfile)
    }
    func main() {
            cli := http.Client{}

            msg := struct {
                Name, Addr string
                Price      float64
            }{
                Name:  "hello",
                Addr:  "beijing",
                Price: 123.56,
            }
            r, w := io.Pipe()
            // 注意这边的逻辑！！
            go func() {
                defer func() {
                    time.Sleep(time.Second * 2)
                    log.Println("encode完成")
                    // 只有这里关闭了，Post方法才会返回
                    w.Close()
                }()
                log.Println("管道准备输出")
                // 只有Post开始读取数据，这里才开始encode，并传输
                err := json.NewEncoder(w).Encode(msg)
                log.Println("管道输出数据完毕")
                if err != nil {
                    log.Fatalln("encode json failed:", err)
                }
            }()
            time.Sleep(time.Second * 1)
            log.Println("开始从管道读取数据")
            resp, err := cli.Post("http://localhost:9999/json", "application/json", r)

            if err != nil {
                log.Fatalln(err)
            }
            log.Println("POST传输完成")

            body := resp.Body
            defer body.Close()

            if body_bytes, err := ioutil.ReadAll(body); err == nil {
                log.Println("response:", string(body_bytes))
            } else {
                log.Fatalln(err)
            }
    }

输出如下：

    main.go:35: 管道准备输出
    main.go:44: 开始从管道读取数据
    main.go:38: 管道输出数据完毕
    main.go:31: encode完成
    main.go:50: POST传输完成
    main.go:56: response: {"Name":"hello","Addr":"beijing","Price":123.56}

可以看到，通过Pipe，我们可以方便的链接输入和输出，只要我们能正确理解整个过程的逻辑。有了它，我们终于可以甩去讨厌的中间缓存了，同时也提高了系统的稳定性。

### 服务器端代码

为了方便大家调试，以下是服务器端代码：

    package main

    import (
               "encoding/json"
               "io/ioutil"
               "log"
               "net/http"
    )

    func init() {
               log.SetFlags(log.Lshortfile)
    }

    func main() {
               http.HandleFunc("/json", handleJson)

               http.ListenAndServe(":9999", nil)
    }

    func handleJson(resp http.ResponseWriter, req *http.Request) {
               if req.Method == "POST" {
                   body := req.Body
                   defer body.Close()
                   body_bytes, err := ioutil.ReadAll(body)
                   if err != nil {
                       log.Println(err)
                       resp.Write([]byte(err.Error()))
                       return
                   }
                   j := map[string]interface{}{}
                   if err := json.Unmarshal(body_bytes, &j); err != nil {
                       log.Println(err)
                       resp.Write([]byte(err.Error()))
                       return
                   }
                   resp.Write(body_bytes)
               } else {
                   resp.Write([]byte("请使用post方法!"))
               }
    }

