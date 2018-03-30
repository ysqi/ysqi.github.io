
---
date: 2016-12-31T11:34:49+08:00
title: "golang在windows平台使用zmq"
description: ""
disqus_identifier: 1485833689140403993
slug: "golangzai-windowsping-tai-shi-yong-zmq"
source: "https://segmentfault.com/a/1190000000624206"
tags: 
- zeromq 
- golang 
topics:
- 编程语言与开发
---

zmq官方推荐的golang库,guthub地址是<http://github.com/pebbe/zmq4>\
测试代码就不发了,上面的地址有具体示例,\
前几天碰到的问题是在windows 7 64位系统环境下go get
github.com\\pebbe\\zmq4的时候无法完成\
最开始可能是提示SOCKET未定义,\
查看这个包的代码可以发现这套库使用了cgo,这是需要gcc等一些环境支持了

不推荐cygwin,因为我测试的时候,在这套环境下仍然无法编译成功\
这时需要安装mingw,注意系统是32还是64的,一定要安装对应的版本,否则无法编译成功

环境装好后编译,再报错找不到zmq.h\
去zmq安装目录\\include文件夹下复制.h头文件\
放到mingw64\\lib\\gcc\\x86\_64-w64-mingw32\\4.9.1\\include文件夹下,\
目录可能不同,只要在mingw安装目录搜索.h文件,查看目录就知道了

再次编译报错,提示\
ld.exe cannot find -lzmq\
这是缺少zmq库的意思,去zmq安装目录/lib文件夹下\
复制libzmq-v120-mt-gd-4\_0\_4.lib到mingw64\\x86\_64-w64-mingw32\\lib目录下\
改名为zmq.lib即可\
zmq安装目录lib文件夹下有好多个lib,具体使用哪一个zmq官方网站有说明.\
请见<http://zeromq.org/distro:microsoft-windows>

再次编译,即可成功,\
在%GOPATH%\\pkg\\windows\_amd64\\github.com\\pebbe目录下就能看到编译好的zmq4.a文件了

