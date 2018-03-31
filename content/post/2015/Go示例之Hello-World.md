---
date: 2013-01-22T08:33:32
title: Go示例之Hello World
slug: golang-demo-hello_world
categories:
    - 编程语言与开发 
tags:
    - Golang
    - 教程
disqus_identifier: 100019
---

这个Golang示例系列篇旨在给新手提供入门指导，通过直观的代码示例来学习Golang。
本篇我们的第一个程序，都是按照国际惯例，打印“Hello,World” 通过直观的代码示例来学习Golang。

<!--more-->

下面是示例代码。

拷贝下面代码到文件`hello-world.go`

```Go
/*
* @Author: 虞双齐
* @Date:   2015-09-22 08:41:54
* @Last Modified by:   虞双齐
* @Last Modified time: 2015-09-22 08:42:32
* @Blog:	www.superzhu.com
 */

package main

import "fmt"

func main() {
	fmt.Println("hello ,世界")
}

```
点击可以执行运行该DEMO,http://dwz.cn/1KP7A1

### 如何运行Go

在开发调式阶段我们可以直接执行 `go run`来运行程序，但有时候我们希望编译程序，以便发布到生产环境，此时通过`go build`编译程序，便可直接运行。
如下：
```shell
➜  hello-world git:(master) ✗ go run hello-world.go                                                                                                                                                                                                             
hello ,世界                                                                                                                                                                                                                                                     
➜  hello-world git:(master) ✗ go build hello-world.go                                                                                                                                                                                                           
➜  hello-world git:(master) ✗ ls                                                                                                                                                                                                                                
hello-world  hello-world.go                                                                                                                                                                                                                                     
➜  hello-world git:(master) ✗ ./hello-world                                                                                                                                                                                                                     
hello ,世界                                                                                                                                                                                                                                                     
➜  hello-world git:(master) ✗                                                                                                                                                                                                                                   
                                                  
```

上面就是Go学习开篇之旅！
