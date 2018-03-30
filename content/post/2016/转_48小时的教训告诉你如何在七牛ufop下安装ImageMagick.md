
---
date: 2016-12-31T11:32:58+08:00
title: "48小时的教训告诉你如何在七牛ufop下安装ImageMagick"
description: ""
disqus_identifier: 1485833578060282820
slug: "48xiao-shi-de-jiao-xun-gao-su-ni-ru-he-zai-qi-niu-ufopxia-an-zhuang-ImageMagick"
source: "https://segmentfault.com/a/1190000007265753"
tags: 
- 七牛云存储 
- golang 
topics:
- 编程语言与开发
---

背景
----

故事是这样的：

公司系统里面有一个服务是 PDF2JPG (实际上应该是 PPT2JPG，
只是PPT2PDF这一步骤我们利用七牛的云服务来完成。)
早期的时候我们根据系统负载已经实际情况采用了 crontab
来定时获取数据库需要转换的数据。所以这个服务的QPS是 1/60
QPS。是的，你没看错就是这么低，但是用起来还好，毕竟用户容量不大。

但是随着用户规模的增大，我们不得不正视一个问题:就是如何满足用户转换的动态需求
(用户的需求爆发点一般爆发在晚上6点左右）。正好七牛的
[UFOP](http://o9gnz92z5.bkt.clouddn.com/article/dora/ufop/ufop-introduction.html)
能满足我们的需求。而且还有免费的额度。

        内存    CPU    系统盘    按需￥/每小时    月计
        M0C1    512MB    1 Core    10GB    0.088    63.36
        M1C2    1GB    2 Core    10GB    0.176    126.72
        M4C8    4GB    8 Core    10GB    0.650    468
        M8C12    8GB    12 Core    10GB    1.300    936
    注意：M0C1的情况下，标准用户 和 高级用户 ，有 750 小时/月的免费额度。

**我们的规划是这样的 在闲时只开启一个 M1C2 实例， 忙时再开启两个 M1C2
实例。这样既保证有效性也能保证经济性**

实施
----

**写七牛的代码，一定不能只看官方网站的文档，一定要找他们 github 上的项目
[qiniu-ufop-go](https://github.com/qiniudemo/qiniu-ufop-go)
、[ufop-golang](https://github.com/qiniu/ufop-golang)。 不过即使 github
上的也不能全信，有些也是过期的。**

只要你记住以上这个，那么写代码根本不算什么！

部署
----

很多人以为是最难的代码都写了，那么部署应该是分分钟的问题。那就看一下下面的血泪图吧。

七牛采用的是 docker 的方案，自建了更新源，隔离了外网链接。所以安装一个
ImageMagick 别提有多疼了！

首先需要安装 libpng ， 接着安装 ghostscript ，然后再安装
ImageMagick，可是 ImageMagick 死活都找不到 libpng 的路径，所以没办法支持
png。 你必须得先安装 pkg-config 才行。

完整的 ufop.yaml 代码如下。

    image: ubuntu
    build_script:
     - echo building...
     - mv $RESOURCE/* ./
     - sudo ldconfig /usr/local/lib
     - tar zxvf pkg-config-0.29.tar.gz > /dev/null
     - cd pkg-config-0.29 && ./configure --with-internal-glib && make > /dev/null 2>&1 && sudo make install > /dev/null
     - tar zxvf libpng-1.6.24.tar.gz > /dev/null
     - cd libpng-1.6.24 && ./configure > /dev/null && make > /dev/null && sudo make install > /dev/null
     - tar zxvf ghostscript-9.15.tar.gz > /dev/null
     - cd ghostscript-9.15 && ./configure > /dev/null && make > /dev/null 2>&1 && sudo make install > /dev/null
     - tar -zxvf ImageMagick.tar.gz > /dev/null
     - /usr/local/bin/pkg-config --list-all
     - cd ImageMagick-7.0.3-4/ && ./configure  --enable-shared=yes > /dev/null && make > /dev/null && sudo make install > /dev/null
     - sudo ldconfig /usr/local/lib
     - echo "convert -list format"
     - chmod +x ./run
    run: ./run qufop.conf

附上结果图：



