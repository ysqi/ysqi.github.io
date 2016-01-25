---
title: MAC下快速安装Wget
slug: install-wget-in-mac
date: 2015-08-15T08:45:07
topics:
    - 软件
tags:
    - mac
    - wget
    - 教程
disqus_identifier: 100023
---

wget是Linux下非常好用的下载工具，何奈Mac下默认不带此命令，仅有curl，下面记录上老虞在Mac下安装Wget的过程。


### 安装Wget前提条件

Mac已安装Xcode，我相信Mac下的开发人员必然会安装Xcode

### 下载安装包

 GUN  Wget    FTP下载地址:[http://www.gnu.org/software/wget/](http://www.gnu.org/software/wget/)

 我下载的是1.10.1版本。

### 解压Wget压缩包
```
tar zxvf wget-1.10.1.tar.gz
x wget-1.10.1/
x wget-1.10.1/m4/
x wget-1.10.1/m4/lib-ld.m4
x wget-1.10.1/m4/lib-link.m4
x wget-1.10.1/m4/lib-prefix.m4
x wget-1.10.1/m4/wget.m4
x wget-1.10.1/src/
x wget-1.10.1/src/alloca.c
.....
```

### 进入Wget目录
```
cd wget-1.10.1
```

### 执行命令安装Wget
```
#检查
./configure
#编译
make
#安装
sudo make install
```

### Mac成功安装Wget检查

+ 执行安装命令后，最后一行会显示已安装到bin目录
```
/usr/bin/install -c -m 644 wget.1 /usr/local/man/man1/wget.1
```

+ 直接输入Wget命令
```
wget -V
GNU Wget 1.10.1
```

到此已完成Mac下安装Wget命令，你可以尝试下载网页或者文件，如：
```
wget www.superzhu.com
wget ftp://ftp.gnu.org/gnu/wget/wget-1.10.1.tar.gz
```
