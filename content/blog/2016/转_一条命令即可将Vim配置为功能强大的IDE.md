
---
date: 2016-12-31T11:33:02+08:00
title: "一条命令即可将Vim配置为功能强大的IDE"
description: ""
disqus_identifier: 1485833582712869227
slug: "yi-tiao-ming-ling-ji-ke-jiang--Vim-pei-zhi-wei-gong-neng-jiang-da-de--IDE"
source: "https://segmentfault.com/a/1190000007036524"
tags: 
- linux 
- python 
- golang
- vim 
topics:
- 编程语言与开发
---

一条命令即可将 `Vim` 配置为功能强大的 `C/C++` `IDE` 。包括安装不太方便的
`YouCompleteMe` 插件也是自动安装，并且会自动从官网下载最新版本的
`libclang`，然后编译 `YouCompleteMe` 插件需要的 `ycm_core library`
，这或许是目前为止安装 `YouCompleteMe` 插件最简单的姿势。

安装：

    curl -o - https://raw.githubusercontent.com/HmyBmny/vimrc/master/install-vim-plugins | sh

部分插件的使用需要安装一些依赖，诸如 `ctags`
之类的，具体请参考：<https://github.com/HmyBmny/vimrc>

支持所有 `Linux` 平台， `Mac` 没试过，默认是 `C/C++`, 如果想用来开发
`Python`， `Go` 或者其它的语言，只需要找到相应的 `Vim`
插件并将仓库名加到 `.vimrc` 文件即可。

开发 `Python` 只需将下面的代码添加到 `.vimrc` 文件

    Plug `klen/python-mode` 

在终端运行 `vim :PlugInstall +qall` 安装插件后配置完成。

开发 `Go` 只需将下面的代码添加到 `.vimrc` 文件

    Plug `fatih/vim-go`

在终端运行 `vim :PlugInstall +qall` 安装插件后配置完成。

