
---
date: 2016-12-31T11:33:22+08:00
title: "golang并发编程的两种限速方法"
description: ""
disqus_identifier: 1485833602099646636
slug: "golangbing-fa-bian-cheng-de-liang-chong-xian-su-fang-fa"
source: "https://segmentfault.com/a/1190000005944664"
tags: 
- golang 
categories:
- 编程语言与开发
---

引子
----

golang提供了goroutine快速实现并发编程，在实际环境中，如果goroutine中的代码要消耗大量资源时（CPU、内存、带宽等），我们就需要对程序限速，以防止goroutine将资源耗尽。\
以下面伪代码为例，看看goroutine如何拖垮一台DB。假设userList长度为10000，先从数据库中查询userList中的user是否在数据库中存在，存在则忽略，不存在则创建。

    //不使用goroutine，程序运行时间长，但数据库压力不大
    for _,v:=range userList {
        user:=db.user.Get(v.ID)
        if user==nil {
            newUser:=user{ID:v.ID,UserName:v.UserName}
            db.user.Insert(newUser)
        }
    }

    //使用goroutine，程序运行时间短，但数据库可能被拖垮
    for _,v:=range userList {
        u:=v
        go func(){
            user:=db.user.Get(u.ID)
            if user==nil {
                newUser:=user{ID:u.ID,UserName:u.UserName}
                db.user.Insert(newUser)
            }
        }()
    }
    select{}

在示例中，DB在1秒内接收10000次读操作，最大还会接受10000次写操作，普通的DB服务器很难支撑。针对DB，可以在连接池上做手脚，控制访问DB的速度，这里我们讨论两种通用的方法。

方案一
------

在限速时，一种方案是丢弃请求，即请求速度太快时，对后进入的请求直接抛弃。

### 实现

实现逻辑如下：

    package main

    import (
        "sync"
        "time"
    )

    //LimitRate 限速
    type LimitRate struct {
        rate     int
        begin    time.Time
        count    int
        lock     sync.Mutex
    }

    //Limit Limit
    func (l *LimitRate) Limit() bool {
        result := true
        l.lock.Lock()
        //达到每秒速率限制数量，检测记数时间是否大于1秒
        //大于则速率在允许范围内，开始重新记数，返回true
        //小于，则返回false，记数不变
        if l.count == l.rate {
            if time.Now().Sub(l.begin) >= time.Second {
                //速度允许范围内，开始重新记数
                l.begin = time.Now()
                l.count = 0
            } else {
                result = false
            }
        } else {
            //没有达到速率限制数量，记数加1
            l.count++
        }
        l.lock.Unlock()

        return result
    }

    //SetRate 设置每秒允许的请求数
    func (l *LimitRate) SetRate(r int) {
        l.rate = r
        l.begin = time.Now()
    }

    //GetRate 获取每秒允许的请求数
    func (l *LimitRate) GetRate() int {
        return l.rate
    }

### 测试

下面是测试代码：

    package main

    import (
        "fmt"
    )

    func main() {
        var wg sync.WaitGroup
        var lr LimitRate
        lr.SetRate(3)
        
        for i:=0;i<10;i++{
            wg.Add(1)
                go func(){
                    if lr.Limit() {
                        fmt.Println("Got it!")//显示3次Got it!
                    }            
                    wg.Done()
                }()
        }
        wg.Wait()
    }

运行结果

    Got it!
    Got it!
    Got it!

只显示3次Got it!，说明另外7次Limit返回的结果为false。限速成功。

方案二
------

在限速时，另一种方案是等待，即请求速度太快时，后到达的请求等待前面的请求完成后才能运行。这种方案类似一个队列。

### 实现

    //LimitRate 限速
    type LimitRate struct {
        rate       int
        interval   time.Duration
        lastAction time.Time
        lock       sync.Mutex
    }

    //Limit 限速
    package main

    import (
        "sync"
        "time"
    )

    func (l *LimitRate) Limit() bool {
        result := false
        for {
            l.lock.Lock()
            //判断最后一次执行的时间与当前的时间间隔是否大于限速速率
            if time.Now().Sub(l.lastAction) > l.interval {
                l.lastAction = time.Now()
                    result = true
                }
            l.lock.Unlock()
            if result {
                return result
            }
            time.Sleep(l.interval)
        }
    }

    //SetRate 设置Rate
    func (l *LimitRate) SetRate(r int) {
        l.rate = r
        l.interval = time.Microsecond * time.Duration(1000*1000/l.Rate)
    }

    //GetRate 获取Rate
    func (l *LimitRate) GetRate() int {
        return l.rate 
    }

### 测试

    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    func main() {
        var wg sync.WaitGroup
        var lr LimitRate
        lr.SetRate(3)
        
        b:=time.Now()
        for i := 0; i < 10; i++ {
            wg.Add(1)
            go func() {
                if lr.Limit() {
                    fmt.Println("Got it!")
                }
                wg.Done()
            }()
        }
        wg.Wait()
        fmt.Println(time.Since(b))
    }

运行结果

    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    Got it!
    3.004961704s

与方案一不同，显示了10次Got
it!但是运行时间是3.00496秒，同样每秒没有超过3次。限速成功。

改造
----

回到最初的例子中，我们将限速功能加进去。这里需要注意，我们的例子中，请求是不能被丢弃的，只能排队等待，所以我们使用方案二的限速方法。

    var lr LimitRate//方案二
    //限制每秒运行20次，可以根据实际环境调整限速设置，或者由程序动态调整。
    lr.SetRate(20)

    //使用goroutine，程序运行时间短，但数据库可能被拖垮
    for _,v:=range userList {
        u:=v
        go func(){
            lr.Limit()
            user:=db.user.Get(u.ID)
            if user==nil {
                newUser:=user{ID:u.ID,UserName:u.UserName}
                db.user.Insert(newUser)
            }
        }()
    }
    select{}

如果您有更好的方案欢迎交流与分享。

**内容为作者原创，未经允许请勿转载，谢谢合作。**

------------------------------------------------------------------------

关于作者：\
Jesse，目前在Joygenio工作，从事golang语言开发与架构设计。\
正在开发维护的产品：[www.botposter.com](http://www.botposter.com)

