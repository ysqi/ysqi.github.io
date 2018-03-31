
---
date: 2016-12-31T11:32:59+08:00
title: "mac下vim&#43;golang开发环境搭建"
description: ""
disqus_identifier: 1485833579775233057
slug: "mac-xia--vim&#43;golang-kai-fa-huan-jing-da-jian"
source: "https://segmentfault.com/a/1190000007204020"
tags: 
- golang 
- vim 
categories:
- 编程语言与开发
---

今天费了好大劲搞定了mac 下 vim golang 的开发环境，mark 一下\
基本参照
[http://studygolang.com/articl...](http://studygolang.com/articles/1785)\
其中有几点需要注意一下

1.  我在mac下安装，目前mac默认的vim
    version是7.3，无法安装YCM（要求7.4+），所以先安装vim:
    `brew install vim`。安装后将安装后的bin path 添加到系统Path 里\
    `export PATH=$PATH:“your vim bin path”`

2.  YCM的安装最好参照
    [https://github.com/Valloric/Y...](https://github.com/Valloric/YouCompleteMe#mac-os-x)\
    其中安装时需要python的若干安装包 `pip install future`,
    `pip install requests`，其他不太记得了。还需要安装
    `brew install cmake`. 然后执行`./install.py --gocode-completer`

3.  使用ultisnips时，研究了好久才明白如何使用它进行展开，且看我的.vimrc文件

<!-- -->

    " YCM settings
    let g:ycm_key_list_select_completion = ['', '']
    let g:ycm_key_list_previous_completion = ['']
    let g:ycm_key_invoke_completion = ''

    " UltiSnips setting
    let g:UltiSnipsExpandTrigger="<Tab>"
    let g:UltiSnipsJumpForwardTrigger="<c-j>"
    let g:UltiSnipsJumpBackwardTrigger="<c-k>"

YCM和UltiSnips的tab会冲突，但是原班照抄帖子中的设置会报错 no mapping
之类的，查阅了网上的一些资料，将设置改成如上。而在使用的时候，键入im
会出现一些提示
`import <snip> import (...)`选中这个直接按tab不会展开，点击确定，会补全import,
此时以上的那个提示还存在，此时按下tab键会进行展开，如下：

    import (
        "package"
    )

至此，golang 环境配置完毕

最后，关于mac下使用gdb调试程序，请参考\
[http://studygolang.com/articl...](http://studygolang.com/articles/1250)\
[http://www.time-track.cn/inst...](http://www.time-track.cn/install-gdb-in-mac.html)

