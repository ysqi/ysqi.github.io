
---
date: 2017-03-13T08:39:17+08:00
title: "Go配置文件热更新"
description: ""
disqus_identifier: 1489365557874590626
slug: "golangpei-zhi-wen-jian-re-geng-xin"
source: "https://segmentfault.com/a/1190000008487440"
tags: 
- golang 
categories:
- 编程语言与开发
---

配置文件热更新是服务器程序的一个基本功能，通过热更新可以不停机调整程序的配置，特别是在生产环境可以提供极大的便利，比如发现log打得太多了可以动态调高日志等级，业务逻辑参数变化，甚至某个功能模块的开关等都可以动态调整。

每种语言都有自己的热更新实现方式，在golang里面我看到了有人采用了一种错误的实现方式，如下：

    type Config struct {
        Test1 string `json:"Test1"`
        Test2 int `json:"Test2"`
    }

    var (
        config *Config
    )

    func loadConfig() {
        f, err := ioutil.ReadFile("config.json")
        if err != nil {
            fmt.Println("load config error: ", err)
        }
        err = json.Unmarshal(f, &config)
        if err != nil {
            fmt.Println("Para config failed: ", err)
        }

    }
    func init() {
        loadConfig()
        fmt.Println("Load config: ", *config)
        s := make(chan os.Signal, 1)
        signal.Notify(s, syscall.SIGUSR2)
        go func() {
            for {
                <-s
                loadConfig()
                fmt.Println("ReLoad config: ", *config)
            }
        }()

    }

这种方式，的问题就在于config可能被多个routine同时访问，那么loadcofig的时候直接在config上做解析是有问题的。在上面例子中，还是用的最简单的json配置，如果是其他更复杂的配置形式，如xml，需要自己写解析函数（而不是一个json.Unmarshal）的时候更容易放大问题。

可以采用更新的时候对config加锁来解决这个问题：

    package main

    import (
        "encoding/json"
        "fmt"
        "io/ioutil"
        "log"
        "os"
        "os/signal"
        "sync"
        "syscall"
    )

    //用json配置测试
    type Config struct {
        Test1 string `json:"Test1:`
        Test2 int    `json:"Test1:`
    }

    var (
        config     *Config
        configLock = new(sync.RWMutex)
    )

    func loadConfig() bool {
        f, err := ioutil.ReadFile("config.json")
        if err != nil {
            fmt.Println("load config error: ", err)
            return false
        }

        //不同的配置规则，解析复杂度不同
        temp := new(Config)
        err = json.Unmarshal(f, &config)
        if err != nil {
            fmt.Println("Para config failed: ", err)
            return false
        }

        configLock.Lock()
        config = temp
        configLock.Unlock()
        return true
    }

    func GetConfig() *Config {
        configLock.RLock()
        defer configLock.RUnlock()
        return config
    }

    func init() {
        if !loadConfig() {
            os.Exit(1)
        }

        //热更新配置可能有多种触发方式，这里使用系统信号量sigusr1实现
        s := make(chan os.Signal, 1)
        signal.Notify(s, syscall.SIGUSR1)
        go func() {
            for {
                <-s
                log.Println("Reloaded config:", loadConfig())
            }
        }()
    }

    func main() {
        select {}
    }

因为热加载的时候，可能有很多未知的routine在使用config，通过切换配置的时候加锁可以保证多个routine对config的正确操作，不过需要注意的是，获取配置文件的时候也需要加锁。

不知道在golang里面，有没有不加锁的方案可以实现配置热更新？

