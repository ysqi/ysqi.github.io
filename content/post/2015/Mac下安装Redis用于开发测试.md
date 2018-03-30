---
title: Mac下安装Redis用于开发测试
slug: install-redis-in-mac
date: 2015-08-15T09:12:01
topics:
    - 软件
tags:
    - Redis
    - Mac
    - 教程
disqus_identifier: 100022
---

Redis作为NB的No-SQL数据，向往已久，现在在做一个网站，计数类的需求比较大，所以准使用Redis实现。下面记录下虞双齐在Mac下开发程序使用Redis的安装过程。

### 下载Redis安装包
上一篇文章我说了[如何在Mac下快速安装Wget](/2015081501.html)，因此下载Redis安装包就可以利用wget命令了。

+ 首先到下载列表看看最新的版本是什么

 国内快速的查看Redis地址： http://www.redis.cn/download.html

 我现在使用的最新版本是3.0.3

+ 下载安装包

```
cd /tmp
wget http://download.redis.io/releases/redis-3.0.3.tar.gz
```

### 解压Redis安装包
```
tar -zxf redis-3.0.3.tar.gz
```

### 进入目录执行安装命令
```
cd redis-3.0.3
#编译
make
#安装
sudo make install
```

### 检查是否已在Mac下成功安装Redis
```
redis-server -v
```
输出版本信息
```
Redis server v=3.0.3 sha=00000000:0 malloc=libc bits=64 build=abcd09569d8d24f9
```

到此已在Mac下安装Redis ，下一遍告诉大家如何让Redis后台自动运行。
