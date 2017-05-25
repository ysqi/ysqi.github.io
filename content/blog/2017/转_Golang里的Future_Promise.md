
---
date: 2017-05-24T09:17:32+08:00
title: "Golang里的Future_Promise"
description: ""
disqus_identifier: 1495588652891557407
slug: "Golangli-de-Future_Promise"
source: "https://segmentfault.com/a/1190000009002323"
tags: 
- future 
- promise 
- golang 
topics:
- 编程语言与开发
---

现如今，应用执行时最普遍存在的瓶颈就是网络请求了。网络请求只要几毫秒，但是等到返回却要百倍的时间。所以，如果你执行多个网络请求，让他们都并行执行就是减少延迟最好的选择了。[Future/Promise](https://en.wikipedia.org/wiki/Futures_and_promises)就是实现这一目的的手段之一。

一个Future就是说“将来”你需要某些东西（一般就是一个网络请求的结果），但是你现在就要发起这样的请求，并且这个请求会异步执行。或者换一个说法，你需要在后台执行一个异步请求。

Future/Promise模式在多种语言都有对应的实现。比如ES2015就有Promise和async-await，Scala内置了Future，最后在Golang里有goroutine和channel可以实现类似的功能。下面给出一个简单的实现。

    //RequestFuture, http request promise.
    func RequestFuture(url string) <-chan []byte {
        c := make(chan []byte, 1)
        go func() {
            var body []byte
            defer func() {
                c <- body
            }()

            res, err := http.Get(url)
            if err != nil {
                return
            }
            defer res.Body.Close()

            body, _ = ioutil.ReadAll(res.Body)
        }()

        return c
    }

    func main() {
      future := RequestFuture("https://api.github.com/users/octocat/orgs")
      body := <-future
      log.Printf("reponse length: %d", len(body))
    }

`RequestFuture`方法理科返回一个channel，这个时候http请求还在一个goroutine后台异步运行。main方法可以继续执行其他的代码，比如触发其他的`Future`等。当需要结果的时候，我们需要从channel里读取结果。如果http请求还没有返回的话就会阻塞当前的goroutine，知道结果返回。

然而，以上的方法还有一点局限。错误无法返回。在上面的例子里，如果http请求出现错误的话，body的值会是nil/empty。但是，由于channel只能返回一个值，你需要创建一个单独的struct来包装两个返回的结果。

修改以后的结果：

    // RequestFutureV2 return value and error
    func RequestFutureV2(url string) func() ([]byte, error) {
        var body []byte
        var err error

        c := make(chan struct{}, 1)
        go func() {
            defer close(c)

            var res *http.Response
            res, err = http.Get(url)
            if err != nil {
                return
            }

            defer res.Body.Close()
            body, err = ioutil.ReadAll(res.Body)
        }()

        return func() ([]byte, error) {
            <-c
            return body, err
        }
    }

这个方法返回了两个结果，解决了第一个方法的局限性问题。使用的时候是这样的：

    func main() {
        futureV2 := RequestFutureV2("https://api.github.com/users/octocat/orgs")

        // not block
        log.Printf("V2 is this locked again")

        bodyV2, err := futureV2() // block
        if err == nil {
            log.Printf("V2 response length %d\n", len(bodyV2))
        } else {
            log.Printf("V2 error is %v\n", err)
        }
    }

上面的修改带来的好处就是`futureV2()`方法的调用可以是多次的。并且都可以返回同样的结果。

但是，如果你想用这个方法实现很多不同的异步功能，你需要写很多的额外的代码。我们可以写一个util方法来克服这个困难。

    // Future boilerplate method
    func Future(f func() (interface{}, error)) func() (interface{}, error) {
        var result interface{}
        var err error

        c := make(chan struct{}, 1)
        go func() {
            defer close(c)
            result, err = f()
        }()

        return func() (interface{}, error) {
            <-c
            return result, err
        }
    }

调用`Future`方法的时候会执行房里的很多channel方面的小技巧。为了能够达到通用的目的有一个从`[]buyte`-\>`interface{}`-\>`[]byte`的类型转换。如果出现错的话会引发一个运行时的*panic*。

原文：
[http://labs.strava.com/blog/f...](http://labs.strava.com/blog/futures-in-golang/)

Stay tuned to my next episode

