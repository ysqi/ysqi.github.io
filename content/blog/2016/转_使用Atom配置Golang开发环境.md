
---
date: 2016-12-31T11:33:49+08:00
title: "使用Atom配置Golang开发环境"
description: ""
disqus_identifier: 1485833629116594239
slug: "shi-yong-Atompei-zhi-Golangkai-fa-huan-jing"
source: "https://segmentfault.com/a/1190000004933373"
tags: 
- go-plus 
- 快捷键 
- golang 
- atom 
topics:
- 编程语言与开发
---

`Golang`作为一种新的编程语言，也有着非常多的IDE，其中个人觉得使用`IntelliJ`开发是一种很不错的选择，但是免费版有功能是缺失的，如果又不想付费购买专业版的，`Atom`就是一种不错的选择。\
`Atom`是更为先进的文本代码编辑器，是由Github打造的下一代编程开发利器，Atom是开源的，美观，各种炫酷效果,还有有各种强大的插件。\
`Atom`上面配置`Golang`的开发环境，使用`Atom`作用IDE,发现效果很不错，下面记录下配置的步骤。\
首先要先安装下[Atom](https://atom.io/),可以直接下载安装。\
安装`Golang`，Mac上面的安装可以参考：<http://www.jianshu.com/p/358cbc939569>

go-plus插件
-----------

`go-plus`是`Atom`上面的一款开源的`Golang`开发环境插件，项目地址:\
<https://github.com/joefitzgerald/go-plus>\
他需要依赖一些的`Golang`工具:

-   `autocomplete-go` ：gocode的代码自动提示

-   `gofmt` ：使用goftm,goimports,goturns

-   `builder-go`: go-install 和go-test,验证代码，给出建议

-   `gometalinet-linter`: goline,vet,gotype的检查

-   `navigator-godef`: godef

-   `tester-goo` : go test

-   `gorename` : rename工具

安装go-plus
-----------

在*Atom*中的`Preference`中可以找到install菜单,输入`go-plus`:

点击:install，就会开始安装`go-plus`，`go-plus`插件会自动安装对应的依赖插件，如果没有安装对应的`Golang`类库可以使用`go get`安装。

快捷键设置
----------

每个人对于自己熟悉的快捷键都不太一样，`Atom`以及其插件对于的快捷键并不是我习惯的快捷键，因此需要对快捷键进行修改，打开`Preference`,选中`Keybindings`：\
可以找到你需要的快捷键的命令点击左边的`copy按钮`，可以拷贝对应的keymap配置，然后点击`your keymap file`可以看到`keymap.cson`,keymap.cson就是自己的`Atom`快捷键配置文件:

    'atom-text-editor[data-grammar~="go"]:not([mini])':
      'alt-r': 'golang:gorename'

可以修改为自己熟悉的快捷键:

    'atom-text-editor[data-grammar~="go"]:not([mini])':
      'cmd-r': 'golang:gorename'

命令行
------

`go-plus`没有提供编译工具,可以在命令行中直接运行`go程序，需要安装`atom-terminal-panel`，在install中直接输入`atom-terminal-panel\`安装。

直接使用快捷键**control+\`**就可以呼出terminal。\
我个人习惯了`fish shell`还是用不惯atom里面的`termianl`，所以尝试了下`Terminal Plus`,对于fish的支持很好，快捷键是`cmd+shift+t`，可以直接呼出`Terminal Plus`。

