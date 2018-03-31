
---
date: 2016-12-31T11:34:16+08:00
title: "godep使用注意"
description: ""
disqus_identifier: 1485833656283909378
slug: "godepshi-yong-zhu-yi"
source: "https://segmentfault.com/a/1190000003781342"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

godep是目前golang主流的包管理工具，众多基于go语言的项目如docker, coreos,
kubernetes等都是使用godep来解决项目包依赖和版本管理问题。作为命令行工具，godep很简单，常用的基本godep
save和godep
restore两个命令就够了，因此一般新手参照官方说明都能快速上手。不过有几个小问题经常会让人疑惑，这里做一下记录和解释：

godep save的机制
================

在未指定包名的情况下，godep会自动扫描当前目录所属包中import的所有外部依赖库（非系统库），并查看其是否属于某个代码管理工具（比如git，hg）。若是，则把此库获取路径和当前对应的revision（commit
id）记录到当前目录Godeps下的Godeps.json，同时，把不含代码管理信息（如.git目录）的代码拷贝到Godeps/\_workspace/src下，用于后继godep
go build等命令执行时固定查找依赖包的路径。

因此，godep save能否成功执行需要有两个要素：

1.  当前或者需扫描的包均能够编译成功：因此所有依赖包事先都应该已经或go
    get或手工操作保存到当前GOPATH路径下

2.  依赖包必须使用了某个代码管理工具（如git，hg）：这是因为godep需要记录revision

godep restore的机制
===================

godep restore执行时，godep会按照Godeps/Godeps.json内列表，依次执行go get
-d -v
来下载对应依赖包到GOPATH路径下，因此，如果某个原先的依赖包保存路径（GOPATH下的相对路径）与下载url路径不一致，比如kuberbetes在github上路径是github.com/kubernetes，而代码内import则是k8s.io，则会导致无法下载成功，也就是说godep
restore不成功。这种只能手动，比如手动创建\$GOPATH/k8s.io目录，然后git
clone。

Godeps目录的作用
================

godep
save时godep把所有依赖包代码从GOPATH路径拷贝到Godeps目录下，并去除代码管理目录。这个用处主要是为了支撑godep
go tool的一系列操作，尤其是git clone了代码库下来后，通常直接用godep go
install
xxx即可完成编译，一定程度上能够缓解golang比较严格的代码路径和包管理带来的烦恼。而在开发使用IDE时，也可以通过把Godeps/\_workspace添加到GOPATH实现代码跳转和编译等功能，比较方便。因此这个设计其实是godep工具的精髓所在。

