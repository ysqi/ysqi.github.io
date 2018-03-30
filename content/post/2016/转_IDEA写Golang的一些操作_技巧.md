
---
date: 2016-12-31T11:32:33+08:00
title: "IDEA写Golang的一些操作_技巧"
description: ""
disqus_identifier: 1485833553148383556
slug: "IDEAxie-Golangde-yi-xie-cao-zuo-_ji-qiao"
source: "https://segmentfault.com/a/1190000008061776"
tags: 
- idea 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

之前一直用vscode来写Golang，直到有人向我推荐了IDEA，便折服于它的强大。在这里分享一些IDEA的操作和技巧（只说Golang，但一些技巧对其他语言同样有效）。

-   Help -&gt; Keymap
    Reference能够打开快捷键映射的PDF文件，方便我们查看\

<!-- -->

-   在类型、函数、变量上CTRL +
    鼠标左键能快速显示它们的使用位置，更好的一点是能够显示出对变量的读和写，这对阅读代码是很大的帮助。不过有一点需要注意，对变量取地址的操作也会判断为读\

<!-- -->

-   给struct添加json tag。在每个元素后连续ALT + SHIFT +
    鼠标左键添加多个光标，输入反引号(\`)和j，此时会弹出窗口，再按下TAB键，所有元素都会补全tag\

<!-- -->

-   CTRL + SHIFT +
    I快速查看函数定义，不需要跳转到定义文件查看后再返回正在编辑的文件，这种感觉不能更爽\

<!-- -->

-   重构，快捷键SHIFT + F6\

<!-- -->

-   ALT + F1在工程栏中展开当前文件的位置\

<!-- -->

-   File
    Watchers插件，设置为当文件保存时调用gofmt等工具格式化代码，或做其他事情\

<!-- -->

-   我们经常要输入一些重复的代码，比如判断err是否为nil。通过Live
    Template解放双手吧（CTRL + J）\



