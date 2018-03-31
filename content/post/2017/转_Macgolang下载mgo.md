
---
date: 2017-02-18T11:13:39+08:00
title: "Macgolang下载mgo"
description: ""
disqus_identifier: 1487387619219648282
slug: "Mac-golangxia-zai-mgo"
source: "https://my.oschina.net/aidelingyu/blog/838211"
tags: 
- golang 
categories:
- 编程语言与开发
---

mgo是第三方提供的golang连接mongodb的库，使用如下命令，进行下载

    go get labix.org/v2/mgo

会出错，说没有安装bzr，bzr是mgo使用的版本控制软件，全名
bazaar，可以在http://wiki.bazaar.canonical.com/Download下载各操作系统的版本。bzr安装好后，执行上述下载命令，可能还是会出错，错误如下

    bzr: ERROR: Couldn't import bzrlib and dependencies.
    Please check the directory containing bzrlib is on your PYTHONPATH.

    Traceback (most recent call last):
      File "/usr/local/bin/bzr", line 102, in <module>
        import bzrlib
    ImportError: No module named bzrlib

上网查了一下是因为bzr使用的是python2.6版本，使用python
-V查看本机安装的2.7的版本，需要降版本。

进入/usr/local/bin/目录，使用vi或vim修改bzr文件，修改第一行：\#!/usr/bin/python 为 \#!/usr/bin/python2.6，保存退出。在使用前面的下载命令，成功下载mgo。由于网速问题，可能要多下几次。

