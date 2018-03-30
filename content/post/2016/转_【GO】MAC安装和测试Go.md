
---
date: 2016-12-31T11:34:21+08:00
title: "【GO】MAC安装和测试Go"
description: ""
disqus_identifier: 1485833661696203358
slug: "【GO】MACan-zhuang-he-ce-shi-Go"
source: "https://segmentfault.com/a/1190000003701079"
tags: 
- golang 
topics:
- 编程语言与开发
---

一、下载安装GO语言包
====================

-   下载地址：\
    <http://download.csdn.net/detail/shuideyidi/7719779>

-   安装说明(按照默认路径安装即可)：\
    <http://blog.csdn.net/delphiwcdj/article/details/11873023>\
    安装成功之后，在terminal运行go，会显示出相关的go命令。

-   环境变量配置：\
    在\~路径下编辑.bashrc，添加export
    PATH=\$PATH:/usr/local/go/bin，然后运行 source .bashrc。

-   测试例子：\
    编辑test.go，然后运行go run test.go即可。

二、配置Sublime Text2/3
=======================

-   下载地址：<http://www.sublimetext.com/3>

-   配置说明：\
    Ctrl+\` 打开命令行，运行：

        import urllib.request,os;pf = 'Package Control.sublime-package';ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) );open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())

淡淡的忧伤。。。不论是2还是3，运行这个安装命令的时候都会卡死。。。不知道是不是网络的问题。\
先接着用vim吧。

