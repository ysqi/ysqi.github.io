
---
date: 2016-12-31T11:33:10+08:00
title: "Golang限速器"
description: ""
disqus_identifier: 1485833590220273255
slug: "Golang-xian-su-qi"
source: "https://segmentfault.com/a/1190000006777319"
tags: 
- golang 
categories:
- 编程语言与开发
---

限速器
======

之前看到这篇[golang并发编程的两种限速方法](https://segmentfault.com/a/1190000005944664)，觉得
sleep
等待的方式不是特别好，唤醒线程的时间比较长。而且1s内的请求只能均匀的到来，如瞬间来
N 个, 那么只有一个能立刻返回，剩下的只能等待。

【修正】\
根据下图的说明，无论是 sleep 还是 chan block, 发生的事情都是 G 和 P
分离，等待下次轮训，再接着执行，所以可能性能是几乎一样的。（这种说法可能不准确，没有深入）\

限速器的作用还是比较重要的，特别是协程使用较多的应用，如果不加限制，可能会OOM。

简单说下思路：默认时间间隔定为1秒，简化问题。如果 调用 Limit() bool
接口，如果一秒内运行次数到达某个值，那么就阻塞， 直到下一个1秒重新计数。

代码
----

    package lib

    import (
        "sync/atomic"
        "time"
    )

    type RateLimiter struct {
        limit  uint64
        count  uint64
        ticker *time.Ticker
        lockCh chan struct{}
    }

    func NewRateLimiter(limit uint64) *RateLimiter {
        ticker := time.NewTicker(time.Second)
        r := &RateLimiter{
            limit:  limit,
            count:  0,
            ticker: ticker,
            lockCh: make(chan struct{}),
        }

        go func() {
            for range ticker.C {
                if r.count > r.limit {
                    select {
                    case <-r.lockCh:
                    default:
                        r.resetCount()
                    }
                }
                
                if r.count > 0 {
                    r.resetCount()                
                }
            }
        }()

        return r
    }

    func (r *RateLimiter) Limit() bool {
        r.addCount(1)

        if r.getCount() > r.limit {
            var s struct{}
            r.lockCh <- s
        }

        return true
    }

    func (r *RateLimiter) addCount(interval uint64) {
        atomic.AddUint64(&r.count, interval)
    }

    func (r *RateLimiter) getCount() uint64 {
        return atomic.LoadUint64(&r.count)
    }

    func (r *RateLimiter) resetCount() {
        atomic.StoreUint64(&r.count, 1)
    }

测试
----

    package lib

    import (
        "log"
        "testing"
        "time"
    )

    func TestLimit(t *testing.T) {
        limiter := NewRateLimiter(3)
        start := time.Now()

        for i := 0; i < 30; i++ {
            if limiter.Limit() {
                log.Printf("i is %d \n", i)
            }
        }

        end := time.Now()

        d := end.Sub(start)
        log.Println("spends seconds: ", d.Seconds())
    }

比较
----

对比了开篇的sleep实现，limiter设为每秒3次，循环30次，用时分别为:

    2016/08/31 14:38:26 my limiter spends seconds:  9.00220346s
    2016/08/31 14:38:35 sleep limiter spends seconds:  9.762009934s

这里sleep limiter多sleep了两次，大约 0.67s, 减去这个值等于9.092009934。

