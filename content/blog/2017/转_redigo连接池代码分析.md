
---
date: 2017-05-24T09:17:35+08:00
title: "redigo连接池代码分析"
description: ""
disqus_identifier: 1495588655287683132
slug: "redigo-lian-jie-chi-dai-ma-fen-xi"
source: "https://segmentfault.com/a/1190000008853173"
tags: 
- redis 
- golang 
topics:
- 编程语言与开发
---

 *结构体分析*
=============

    type Pool struct {

        // 用来创建redis连接的方法
        Dial func() (Conn, error)

        // 如果设置了给func,那么每次p.Get()的时候都会调用改方法来验证连接的可用性
        TestOnBorrow func(c Conn, t time.Time) error

        // 定义连接池中最大连接数（超过这个数会关闭老的链接，总会保持这个数）
        MaxIdle int

        // 当前连接池中可用的链接数.
        MaxActive int

        // 定义链接的超时时间，每次p.Get()的时候会检测这个连接是否超时（超时会关闭，并释放可用连接数）.
        IdleTimeout time.Duration

        // 当可用连接数为0是，那么当wait=true,那么当调用p.Get()时，会阻塞等待，否则，返回nil.
        Wait bool

        // 读写锁控制.
        mu     sync.Mutex
        // 用来条件控制，这里主要是当链接被关闭时，提醒在等待的进程可以使用了，或者可以自行创建了
        cond   *sync.Cond
        // 当前连接池是否已经关闭
        closed bool
        // 当前可用的链接数
        active int

        // 链接存储在一个栈中.
        idle list.List
    }

***连接池关闭方法***
====================

    func (p *Pool) Close() error {
        p.mu.Lock()
        // 获取连接池所有链接栈
        idle := p.idle
        // 重新初始化
        p.idle.Init()
        // 标示已经关闭
        p.closed = true
        // 控制可用连接数
        p.active -= idle.Len()
        // 如果当前有进程正在等待获取的话，则通知可以获取或者自行创建
        if p.cond != nil {
            p.cond.Broadcast()
        }
        p.mu.Unlock()
        // 遍历栈，逐个关闭链接
        for e := idle.Front(); e != nil; e = e.Next() {
            e.Value.(idleConn).c.Close()
        }
        return nil
    }

******释放一个链接
==================

    func (p *Pool) release() {
        // 当链接超时，或者ping不通，或者创建失败，则立即使用链接表示
        p.active -= 1
        // 如果已经有进程在之前等待了，则通知其使用或者自行创建
        if p.cond != nil {
            p.cond.Signal()
        }
    }

***关闭连接***
==============

    func (p *Pool) put(c Conn, forceClose bool) error {
        err := c.Err()
        p.mu.Lock()
        // 如果连接池没有关闭，并且不是强制关闭的
        if !p.closed && err == nil && !forceClose {
            // 把指定的链接放在空闲栈首位
            p.idle.PushFront(idleConn{t: nowFunc(), c: c})
            // 如果栈的长度大于指定长度，则吧最后一个（可能超时）剔除
            if p.idle.Len() > p.MaxIdle {
                c = p.idle.Remove(p.idle.Back()).(idleConn).c
            } else {
                c = nil
            }
        }

        if c == nil {
            //成功放回空闲连接通知其他阻塞的进程
            if p.cond != nil {
                p.cond.Signal()
            }
            p.mu.Unlock()
            return nil
        }
        // 减少active计数（感觉这里可以不用处理，上面如果是替换的话）
        p.release()
        p.mu.Unlock()
        // 关闭连接
        return c.Close()
    }

***获取链接***
==============

    func (p *Pool) get() (Conn, error) {
        p.mu.Lock()
        // 处理旧的链接（检测是否超时）
        if timeout := p.IdleTimeout; timeout > 0 {
            for i, n := 0, p.idle.Len(); i < n; i++ {
                e := p.idle.Back()
                if e == nil {
                    break
                }
                ic := e.Value.(idleConn)
                // 链接创建的时间加上超时时间是否小于当前时间
                if ic.t.Add(timeout).After(nowFunc()) {
                    break
                }
                // 超时链接，从栈中删除
                p.idle.Remove(e)
                // 可用连接数减少，并通知其他等待的进程处理
                p.release()
                p.mu.Unlock()
                // 关闭当前连接
                ic.c.Close()
                p.mu.Lock()
            }
        }
        for {

            // 从连接栈列表中获取一个可用链接
            for i, n := 0, p.idle.Len(); i < n; i++ {
                // 从idle列表前面取连接，那么必然是刚刚使用过的连接
                e := p.idle.Front()
                if e == nil {
                    break
                }
                // 类型断言
                ic := e.Value.(idleConn)
                // 从栈中删除
                p.idle.Remove(e)
                test := p.TestOnBorrow
                p.mu.Unlock()
                // 通过使用校验链接函数检测链接
                if test == nil || test(ic.c, ic.t) == nil {
                    return ic.c, nil
                }
                // 检验出问题，关闭连接
                ic.c.Close()
                p.mu.Lock()
                // 降低可用连接数，并通知其他进程处理
                p.release()
            }

            // 检测连接池是否已经关闭.
            if p.closed {
                p.mu.Unlock()
                return nil, errors.New("redigo: get on closed pool")
            }

            // 如果可用连接数为0 或者小于最大可用可用连接数范围，那么创建
            if p.MaxActive == 0 || p.active < p.MaxActive {
                dial := p.Dial
                p.active += 1
                p.mu.Unlock()
                // 连接redis server
                c, err := dial()
                // 连接失败关闭之
                if err != nil {
                    p.mu.Lock()
                    p.release()
                    p.mu.Unlock()
                    c = nil
                }
                return c, err
            }
            // 如果没有配置等待，那么就返回nil
            if !p.Wait {
                p.mu.Unlock()
                return nil, ErrPoolExhausted
            }
            // 如果配置了等待，那么初始化，并且开始等待，直到有进程通知链接数够了或者可以创建了
            if p.cond == nil {
                p.cond = sync.NewCond(&p.mu)
            }
            p.cond.Wait()
        }
    }

