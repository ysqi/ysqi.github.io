---
date: 2013-04-05T23:31:47
title: 老虞学Golang-常量
slug: ysqi-golang-const
topics:
- 编程语言与开发
tags:
- Go
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100016
---

###常量

常量和C#中的概念相同，在编译期被创建。因为在编译期必须确定其值，因此在声明常量时有一些限制。

+ 其类型必须是：数值、字符串、布尔值
+ 表达式必须是在编译期可计算的
+ 声明常量的同时必须进行初始化，其值不可再次修改


####Doc

http://golang.org/doc/go_spec.html#Constants
http://golang.org/doc/go_spec.html#Constant_expressions
http://golang.org/doc/go_spec.html#Constant_declarations
http://golang.org//doc/go_spec.html#Iota

####语法

const关键字用于声明常量` const [(] 名称 [数据类型] = 表达式 [)]` `const ( 多个常量名称 [数据类型]= 对应的多个表达式 )`

如果定义多行常量而表达式一致时可省略其他行的表达式

声明时如果不指定数据类型，则该常量为无类型常量
```Go
const Pi = 3.14159265358 //float64
Pi=3.1415
```
编译错误: `cannot assign to Pi`, 变量名Pi已经被使用这里是无法再次给Pi赋值的
```Go
const a, b, c = 1, false, "str" //多重赋值
```
一次可以声明多个常量，且同时赋值，其类型可以不一致
```Go
const d = 1 << 2 //需计算的表达式
```
复制可以是一个可以在编译期计算出结果的表达式
```Go
const ( //批量声明
  Monday, Tuesday, Wednesday = 1, 2, 3
  Thursday, Friday, Saturday = 4, 5, 6
 )
 ```
批量声明多个常量
```Go
const e, f float64 = 1, 2 / 1.0
```
声明多个变量时，指定其数据类型，此时 e和f均是float64数据类型
```Go
const g int ,h float64 = 1,2/1.0
```
不能分别指定数据类型，数据类型后面需要跟着 '=',编译错误: `syntax error: unexpected comma, expecting =`

###iota

在Go中使用另一个常量iota计数器,只能在常量的表达式中使用。 iota在const关键字出现时将被重置为0(const内部的第一行之前)，const中每新增一行常量声明将使iota计数一次(iota可理解为const语句块中的行索引)。使用iota能简化定义，在定义枚举时很有用。

```Go
fmt.Println(iota)
```
iota只能在const内部使用。编译错误： undefined: iota

```Go
const a = iota // a=0
const (
  b = iota     //b=0
  c            //c=1
)
```
iota从0开始计数

```Go
const (
  bit00 uint32 = 1 << iota //bit00=1
  bit01                    //bit01=2
  bit02                    //bit02=4
 )
 ```
iota可在表达式中(b=iota也是表达式)

```Go
 const (
  loc0, bit0 uint32 = iota, 1 << iota   //loc0=0,bit0=1
  loc1, bit1                            //loc1=1,bit1=2
  loc2, bit2                            //loc2=2,bit2=4
 )
```

```Go
const (
  e, f, g = iota, iota, iota //e=0,f=0,g=0
)
```
在同一行,iota相同

```Go
const (
    h = iota    //h=0
    i = 100     //i=100
    j           //j=100
    k = iota    //k=3  
)
```

虽然只使用了两次iota，但每新起一行iota都会计数.
