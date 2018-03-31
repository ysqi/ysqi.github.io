
---
date: 2016-12-31T11:33:32+08:00
title: "Golang语法吐漕"
description: ""
disqus_identifier: 1485833612759398578
slug: "Golang-yu-fa-tu-cao"
source: "https://segmentfault.com/a/1190000005336434"
tags: 
- golang 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

func (e JsonEncoder) Encode(obj interface{}) (\[\]byte, error) {

}

从这样一个函数声明来看吧：

1.  类型放变量名后面

        跟所有其他语言相反。不知道哪根筋搭错了，非得逆行。

2.  诡异的类定义

        类没有明显边界，谁知道哪个角落里写了一个类方法？在大型项目里面多人协作的时候可能会有坑。

3.  怪异的 nil

         其他语言大部分都是 null，虽然有点坑，好歹是个代词。nil 除了少敲一个字母，实在怪异。怪异程度堪比 javascript 的 NaN



