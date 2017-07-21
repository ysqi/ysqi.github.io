
---
date: 2017-02-24T08:31:54+08:00
title: "Golang学习摘录四:包"
description: ""
disqus_identifier: 1487896314967989529
slug: "Golangxue-xi-zhai-lu-si-:bao"
source: "http://www.jianshu.com/p/d9f5224be517"
tags: 
- golang 
topics:
- 编程语言与开发
---

包是函数和数据的集合。用 package 关键字定义一个包。文件名不需要与包名
一致。包名的约定是使用小写字符。Go 包可以由多个文件组成,但是使用相同的
package \<name\> 这一行。

    package even
    func Even(i int) bool {
      return i%2==0
    }
    func odd(i int) bool {
      return i%2==1
    }

名称以大写字母起始的是可导出的，可以在包的外部调用。\
共有函数的名称以大写字母开头。\
私有函数的名称以小写字母开头。

    package main
    import ( //导入下面的包
      "event" //本地包even在这里导入
      "fmt"
    )
    func main(){
      i := 5
      fmt.Printf("Is %d event? %v\n", i,event.Even(i))// 调用even包中的函数。访问一个包中的函数的语法是<package>.Function()。
    }

Go程序的命名规则：包名约定小写字母开头；方法名最好使用驼峰式名称，避免使用下划线，方法名应简洁清晰。

包的文档
--------

每个包都应该有包注释,在 package
前的一个注释块。对于多文件包,包注释只需要
出现在一个文件前,任意一个文件都可以。包注释应当对包进行介绍,并提供相关于
包的整体信息。这会出现在 go doc
生成的关于包的页面上,并且相关的细节会一并 显示。\
官方regexp包的例子：

     /*
           The regexp package implements a simple library for
           regular expressions.
           The syntax of the regular expressions accepted is:
           regexp:
               concatenation  '|' concatenation
       */
       package regexp

每个定义（并且导出）的函数应当有一小段文字描述该函数的行为。\
fmt包的例子：

       // Printf formats according to a format specifier and writes to standard
       // output. It returns the number of bytes written and any write error
       // encountered.
       func Printf(format string, a ...interface) (n int, err error)

测试包
------

在Go中为包编写单元测试应当是一种习惯。编写测试需要包含testing包和程序go
test。两者都有良好的文档。\
go test程序调用了所有的测试函数。\
even包没有定义任何测试函数，执行go test，是这样的：

    % go test
    ?    even  [no test files]

在测试文件中定义一个测试来修复这个。测试文件也在包目录中，被命名为\*\_rest.go。这些测试文件同Go程序中的其他文件一样，但是go
test只会执行测试函数。每个测试函数都有相同的标示，他的名字以Test开头：

    func TestXxx(t *testing.T)

编写测试时，需要告诉go test测试是失败还是成功。测试成功则直接返回。当测
试失败可以用下面的函数标记 。

    func (t *T) Fail() // Fail标记测试函数失败，但仍然继续执行。
    func (t *T) FailNow() // FailNow标记测试函数失败，并且中断其执行。当前文件中的其余的测试将被跳过,然后执行下一个文件中的测试。
    func (t *T) Log(args ...interface{}) // Log 用默认格式对其参数进行格式化,与 Print() 类似,并且记录文本到错误日志。
    func (t *T) Fatal(args ...interface{}) // Fatal 等价于 Log() 后跟随 FailNow()。

event\_test.go

    package even // 测试使用与被测试的包使用相同的名称空间
    import "testing"
    func TestEven(t *testing.T) { // 定义测试函数
      if ! Even(2) { 
        t.Log("should be even!") 
        t.Fail() 
      }
    }

常用的包
--------

标准的 Go 代码库中包含了大量的包,并且在安装 Go
的时候多数会伴随一起安装。浏 览 \$GOROOT/src/pkg
目录并且查看那些包会非常有启发。

-   fmt\
    包 fmt 实现了格式化的 I/O 函数,这与 C 的 printf 和 scanf
    类似。格式化短语 派生于 C 。一些短语(%-序列)这样使用:\
    %v\
    默认格式的值。当打印结构时,加号(%+v)会增加字段名; %\#v\
    Go 样式的值表达;\
    %T\
    带有类型的 Go 样式的值表达;
-   io\
    这个包提供了原始的 I/O 操作界面。它主要的任务是对 os 包这样的原始的
    I/O 进 行封装,增加一些其他相关,使其具有抽象功能用在公共的接口上。
-   bufio\
    这个包实现了缓冲的 I/O。它封装于 io.Reader 和 io.Writer
    对象,创建了另 一个对象(Reader 和
    Writer)在提供缓冲的同时实现了一些文本 I/O 的功能。
-   sort\
    sort 包提供了对数组和用户定义集合的原始的排序功能。
-   strconv\
    strconv
    包提供了将字符串转换成基本数据类型,或者从基本数据类型转换为字
    符串的功能。
-   os\
    os 包提供了与平台无关的操作系统功能接口。其设计是 Unix 形式的。
-   sync\
    sync 包提供了基本的同步原语,例如互斥锁。
-   flag\
    flag 包实现了命令行解析。参阅 “命令行参数” 在第 92 页。
-   encoding/json\
    encoding/json 包实现了编码与解码 RFC 4627 [2] 定义的 JSON 对象。
-   html/template\
    数据驱动的模板,用于生成文本输出,例如 HTML。\
    将模板关联到某个数据结构上进行解析。模板内容指向数据结构的元素(通常结
    构的字段或者 map 的键)控制解析并且决定某个值会被显示。模板扫描结构以
    便解析,而 “游标” @ 决定了当前位置在结构中的值。
-   net/http\
    net/http 实现了 HTTP 请求、响应和 URL 的解析,并且提供了可扩展的 HTTP
    服 务和基本的 HTTP 客户端。
-   unsafe\
    unsafe 包包含了 Go
    程序中数据类型上所有不安全的操作。通常无须使用这个。
-   reflect\
    reflect
    包实现了运行时反射,允许程序通过抽象类型操作对象。通常用于处理静
    态类型 interface{} 的值,并且通过 Typeof
    解析出其动态类型信息,通常会返回 一个有接口类型 Type 的对象。
-   os/exec\
    os/exec 包执行外部命令。


