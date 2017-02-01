
---
date: 2016-12-31T11:34:37+08:00
title: "[转载]Golanghotconfigurationreload"
description: ""
disqus_identifier: 1485833677364061805
slug: "[zhuai-zai-]-Golang-hot-configuration-reload"
source: "https://segmentfault.com/a/1190000002492016"
tags: 
- 热加载 
- golang 
topics:
- 编程语言与开发
---

原文：<http://openmymind.net/Golang-Hot-Configuration-Reload/>

Like most, I've always appreciated a software package that lets me
hot-reload the configuration without having to restart. Nginx
immediately comes to mind, as does Postgresql (not for all settings, but
most). This is something I've never done; so I was pretty happy when the
opportunity presented itself.

I'm not sure how anyone else does this, but it's pretty basic stuff.
When you get past the meaningless fact that you're reloading a
configuration, it's just about providing concurrent read and write
access to a shared object. There are a lot of ways to do that and a lot
will depend on your language/framework. In Go, our approach was to use a
read/write mutex.

First, our basic configuration structure, a global (to the package)
variable and a similar lock:

    goimport (
      "os"
      "log"
      "sync"
      "syscall"
      "os/signal"
      "io/ioutil"
      "encoding/json"
    )

    type Config struct {
      Mode string
      CacheSize int
    }

    var (
      config *Config
      configLock = new(sync.RWMutex)
    )

Next we need a function that loads our config:

    gofunc loadConfig(fail bool){
      file, err := ioutil.ReadFile("config.json")
      if err != nil {
        log.Println("open config: ", err)
        if fail { os.Exit(1) }
      }

      temp := new(Config)
      if err = json.Unmarshal(file, temp); err != nil {
        log.Println("parse config: ", err)
        if fail { os.Exit(1) }
      }
      configLock.Lock()
      config = temp
      configLock.Unlock()
    }

Whether we exit on failure depends on whether this is the first time
loading the configuration or if it's a reload. We use a temp variable to
hold our new reference so that we can minimize the duration of our write
lock to the simplist possible code.

Next we need is a way to access the current configuration:

    gofunc GetConfig() *Config {
      configLock.RLock()
      defer configLock.RUnlock()
      return config
    }

Finally we need to do an initial load as well as be able to cause a
reload. To do this, we listen for a signal:

    go// go calls init on start
    func init() {
      loadConfig(true)
      s := make(chan os.Signal, 1)
      signal.Notify(s, syscall.SIGUSR2)
      go func() {
        for {
          <-s
          loadConfig(false)
          log.Println("Reloaded")
        }
      }()
    }

Like I said, not hard but at least a little neat.

