
---
date: 2016-12-31T11:32:52+08:00
title: "使用context实现多个goroutine的依赖管理"
description: ""
disqus_identifier: 1485833572598139071
slug: "shi-yong-contextshi-xian-duo-ge-goroutinede-yi-lai-guan-li"
source: "https://segmentfault.com/a/1190000007531146"
tags: 
- golang 
categories:
- 编程语言与开发
---

解决的问题
----------

在很多实际情况，比如处理网络请求时，我们需要启动多个goroutine来处理不同的逻辑，比如一个主要的goroutine用来响应请求，生成网页，同时它还启动一个子线程用来获取数据库信息，还有一个则写日志等等。正常情况都没有问题，但是一旦出现异常，如何优雅的退出这些子线程，同时释放掉可能占用的资源呢？

context
-------

在golang中，人们发明了context接口处理这种情况。早在14年，这个库就出现了，并且提出了[基于context的并发编程范式](https://blog.golang.org/context)（英文好的同学可以直接撸这篇文章）。\
今年8月[go1.7](https://golang.org/doc/go1.7)发布后，它正式成为了标准库的一员。

如何使用
--------

在golang的context库中，首先定义了context的接口，然后给出了context接口的4种实现：

-   WithCancel(parent Context) (Context, CancelFunc)\
    初始化一个可以被cancel的context，同时把新context对象作为child放入parent的children数组中。当parent终止时，child也会接受到信号。这个过程叫`propagateCancel`

-   WithDeadline(parent Context, deadline time.Time) (Context,
    CancelFunc)\
    同样初始化一个context，除了实现跟`WithCancel`同样的功能外，还增加了一个时间变量，一旦当前时间超过这个deadline，那么这个context以及它的所有子孙都被被cancel。

-   WithTimeout(parent Context, timeout time.Duration) (Context,
    CancelFunc)\
    跟`WithDeadline`类似，如果说`WithDeadline`是一个绝对时间上的限制，那么`WithTimeout`就是一个相对时间的限制

-   WithValue(parent Context, key, val interface{}) Context\
    单纯给parent增加value，不需要`propagateCancel`。value可以用来跨进程、跨api的传递数据，最好是和某个请求相关的参数，不要传递太多大量数据。

所以关键就在于`propagateCancel`，实际工程中，所有context共同组成了一个依赖树，他们都继承自一个祖先。一旦parent被cancel，就会通过`propagateCancel`递归的传播给下面的所有子孙。可以看出，context就好比信使，或者说通讯协议，通过遵循context接口构建的这个框架，能够保证子线程及时获得与他相关的父线程的状态，从而由子线程根据情况作出反应。至于怎么反应，就取决于各位码农的能力和搬砖当时的心情了。。。

另外，golang有一套静态分析工具可以分析context的传播过程，所以为了方便这个工具的使用，实际使用中有几个规定：

-   不要把context作为struct内部变量使用，而是把它和其他变量一块作为参数传入下一个函数。

-   context变量需要作为函数的第一个参数传入，命名一般为`ctx`

具体例子
--------

这个例子来源于[基于context的并发编程范式](https://blog.golang.org/context)，但是为了符合国情我做了些修改：\
包括3部分：

-   server.go\
    主线程，会创建一个server服务器，可以通过`localhost:9090/search`访问。接到请求后，它会创建父context，同时生成一个新goroutine，去fakesrv（本来应该去google上的）上请求数据。

-   google.go\
    替换原来的google网址，改成由fakesrv提供的一个网址。主要就是演示一下context的运行过程，请求fakesrv的工作在一个新goroutine中进行，同时它还有一个访问数据库的操作。如果父context因为timeout超时了，那么对fakesrv和数据库的访问也会终止。在代码中，演示了如何监听context信息的过程。

-   query.go\
    解析url中的query参数

-   fakesrv.go\
    提供[http://localhost:9000/context...](http://localhost:9000/context_demo)供google.go访问。

### mycontext/serve.go

    // The server program issues Google search requests and demonstrates the use of
    // the go.net Context API. It serves on port 8080.
    //
    // The /search endpoint accepts these query params:
    //   q=the Google search query
    //   timeout=a timeout for the request, in time.Duration format
    //
    // For example, http://localhost:8080/search?q=golang&timeout=1s serves the
    // first few Google search results for "golang" or a "deadline exceeded" error
    // if the timeout expires.
    package main

    import (
        "html/template"
        "log"
        "net/http"
        "time"

        "context"
        "mycontext/google"
        "mycontext/query"
    )

    func main() {
        http.HandleFunc("/search", handleSearch)
        log.Fatal(http.ListenAndServe(":9090", nil))
    }

    // handleSearch handles URLs like /search?q=golang&timeout=1s by forwarding the
    // query to google.Search. If the query param includes timeout, the search is
    // canceled after that duration elapses.
    func handleSearch(w http.ResponseWriter, req *http.Request) {
        // ctx is the Context for this handler. Calling cancel closes the
        // ctx.Done channel, which is the cancellation signal for requests
        // started by this handler.
        var (
            ctx    context.Context
            qctx   *query.QueryCtx
            cancel context.CancelFunc
        )
        timeout, err := time.ParseDuration(req.FormValue("timeout"))
        if err == nil {
            // The request has a timeout, so create a context that is
            // canceled automatically when the timeout expires.
            ctx, cancel = context.WithTimeout(context.Background(), timeout)
        } else {
            ctx, cancel = context.WithCancel(context.Background())
        }
        defer cancel() // Cancel ctx as soon as handleSearch returns.
        qctx, err = query.NewQueryCtx(ctx, req)
        if err != nil {
            http.Error(w, "no query", http.StatusBadRequest)
            return
        }

        // Run the Google search and print the results.
        start := time.Now()
        results, err := google.Search(qctx)
        elapsed := time.Since(start)
        if err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        if err := resultsTemplate.Execute(w, struct {
            Results          google.Results
            Timeout, Elapsed time.Duration
        }{
            Results: results,
            Timeout: timeout,
            Elapsed: elapsed,
        }); err != nil {
            log.Print(err)
            return
        }
    }

    var resultsTemplate = template.Must(template.New("results").Parse(`
    <html>
    <head/>
    <body>
      <ol>
      {{range .Results}}
        <li>{{.Title}} - <span>{{.SubTitle}}</span></li>
      {{end}}
      </ol>
      <p>{{len .Results}} results in {{.Elapsed}}; timeout {{.Timeout}}</p>
    </body>
    </html>
    `))

### mycontext/google/google.go

    // Package google provides a function to do Google searches using the Google Web
    // Search API. See https://developers.google.com/web-search/docs/
    //
    // This package is an example to accompany https://blog.golang.org/context.
    // It is not intended for use by others.
    //
    // Google has since disabled its search API,
    // and so this package is no longer useful.
    package google

    import (
        "context"
        "encoding/json"
        "log"
        "mycontext/query"
        "net/http"
        "time"
    )

    // Results is an ordered list of search results.
    type Results []Result

    // A Result contains the title and URL of a search result.
    type Result struct {
        Title, SubTitle string
    }

    // Search sends query to Google search and returns the results.
    func Search(ctx *query.QueryCtx) (Results, error) {
        // Prepare the Google Search API request.
        req, err := http.NewRequest("GET", "http://localhost:9000/context_demo", nil)
        if err != nil {
            return nil, err
        }

        ctx.SetReq(req)
        // Issue the HTTP request and handle the response. The httpDo function
        // cancels the request if ctx.Done is closed.
        var results Results
        err = httpDo(ctx, req, func(resp *http.Response, err error) error {
            if err != nil {
                return err
            }
            defer resp.Body.Close()

            // Parse the JSON search result.
            // https://developers.google.com/web-search/docs/#fonje
            var data struct {
                ResponseData struct {
                    Results []struct {
                        Title, SubTitle string
                    }
                }
            }
            if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
                return err
            }
            for _, res := range data.ResponseData.Results {
                results = append(results, Result{Title: res.Title, SubTitle: res.SubTitle})
            }
            return nil
        })
        // httpDo waits for the closure we provided to return, so it's safe to
        // read results here.
        return results, err
    }

    // httpDo issues the HTTP request and calls f with the response. If ctx.Done is
    // closed while the request or f is running, httpDo cancels the request, waits
    // for f to exit, and returns ctx.Err. Otherwise, httpDo returns f's error.
    func httpDo(ctx *query.QueryCtx, req *http.Request, f func(*http.Response, error) error) error {
        // Run the HTTP request in a goroutine and pass the response to f.
        tr := &http.Transport{}
        client := &http.Client{Transport: tr}
        // WithCancel会在ctx的children中增加cancelDb，这样当
        // ctx 结束的时候，cancelDb也会受到消息
        cancelDb, cancel := context.WithCancel(ctx.Context)
        defer cancel()
        c := make(chan error, 1)
        go func() { c <- f(client.Do(req)) }()
        go func(ctx context.Context) {
            t := time.NewTimer(2 * time.Second)

            select {
            case <-t.C:
                log.Println("db access finished!")
            case <-ctx.Done():
                log.Println("canceld by parent, release resource")
            }
        }(cancelDb)
        select {
        case <-ctx.Done():
            tr.CancelRequest(req)
            <-c // Wait for f to return.
            return ctx.Err()
        case err := <-c:
            return err
        }
    }

### mycontext/query/query.go

    package query

    import (
        "context"
        "fmt"
        "net/http"
    )

    func NewQueryCtx(ctx context.Context, req *http.Request) (*QueryCtx, error) {
        q := req.FormValue("q")
        if q == "" {
            return nil, fmt.Errorf("no query supplied!")
        }
        return &QueryCtx{ctx, q}, nil
    }

    type QueryCtx struct {
        context.Context
        val string
    }

    func (ctx *QueryCtx) SetReq(req *http.Request) {
        q := req.URL.Query()
        q.Set("q", ctx.val)

        req.URL.RawQuery = q.Encode()
    }

### mycontext/fakesrv/main.go

    package main

    import (
        "bytes"
        "encoding/json"
        "fmt"
        "log"
        "math/rand"
        "net/http"
        "strconv"
        "strings"
        "time"
    )

    func init() {
        log.SetFlags(log.Lshortfile)
    }

    type Results struct {
        ResponseData struct {
            Results []Content
        }
    }

    // A Result contains the title and URL of a search result.
    type Content struct {
        Title, SubTitle string
    }

    func main() {

        http.HandleFunc("/context_demo", handleContext)
        http.ListenAndServe(":9000", nil)
    }

    func handleContext(resp http.ResponseWriter, req *http.Request) {
        defer func() {
            if e := recover(); e != nil {
                if msg, ok := e.(string); ok {
                    resp.Write([]byte(msg))
                } else {
                    panic(e)
                }
            }
        }()
        check_error := func(err error, msg string) {
            if err != nil {
                if msg != "" {
                    panic(err.Error() + ":" + msg)
                } else {
                    panic(err.Error())
                }
            }
        }
        if req.Method == "GET" {
            q := req.FormValue("q")
            seg := strings.Split(q, ":")
            if len(seg) < 2 {
                log.Println("query format wrong")
                resp.Write([]byte("query format wrong"))
                return
            }
            title := seg[0]
            num, err := strconv.Atoi(seg[1])
            check_error(err, "")
            rs := Results{}
            for i := 0; i < num; i++ {
                rs.ResponseData.Results = append(rs.ResponseData.Results,
                    Content{fmt.Sprintf("%s %d", title, i), RandomString(20)})
            }
            buff := bytes.NewBuffer(nil)
            err = json.NewEncoder(buff).Encode(rs)
            check_error(err, "")
            time.Sleep(time.Second * 2)
            resp.Write(buff.Bytes())
        } else {
            resp.Write([]byte("请使用get方法!"))
        }
    }
    func RandomString(strlen int) string {
        rand.Seed(time.Now().UTC().UnixNano())
        const chars = "abcdefghijklmnopqrstuvwxyz0123456789"
        result := make([]byte, strlen)
        for i := 0; i < strlen; i++ {
            result[i] = chars[rand.Intn(len(chars))]
        }
        return string(result)
    }

### Makefile

    run:
        go build 
        ./mycontext &
        cd fakesrv && go build && ./fakesrv &

    test:
        @echo "======= test without timeout ======="
        curl localhost:9090/search?q=title:6
        @echo "======= test with timeout 1s ======="
        curl localhost:9090/search?q=title:6\&timeout=1s
        @echo "======= test with timeout 4s ======="
        curl localhost:9090/search?q=title:6\&timeout=4s

### 测试

在命令行运行如下命令，即可看到具体结果\
make run\
make test

