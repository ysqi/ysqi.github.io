---
date: 2017-02-24T08:31:58+08:00
title: "编写地道的Go代码"
description: ""
disqus_identifier: 1487896318515149662
slug: "bian-xie-de-dao-de-Godai-ma"
source: "http://colobu.com/2017/02/07/write-idiomatic-golang-codes/"
tags: 
- golang 
categories:
- 编程语言与开发
---

在阅读本文之前，我先推荐你阅读官方的 [Effective
Go](https://golang.org/doc/effective_go.html)文档，或者是中文翻译版:
[高效Go编程](https://go-zh.org/doc/effective_go.html)，它提供了很多编写标准而高效的Go代码指导，本文不会再重复介绍这些内容。

最地道的Go代码就是Go的标准库的代码，你有空的时候可以多看看Google的工程师是如何实现的。

本文仅作为一个参考，如果你有好的建议和意见，欢迎添加评论。

### 注释

可以通过 `/* …… */` 或者 `// ……`增加注释， `//`之后应该加一个空格。

如果你想在每个文件中的头部加上注释，需要在版权注释和
Package前面加一个空行，否则版权注释会作为Package的注释。
 
```go
 // Copyright 2009 The Go Authors. All rights reserved.
 // Use of this source code is governed by a BSD-style
 // license that can be found in the LICENSE file.
 
/*Package net provides a portable interface for network I/O,
  includingTCP/IP, UDP, domain name resolution, and Unix domain sockets.......
*/
package net......        
```

注释应该用一个完整的句子，注释的第一个单词应该是要注释的指示符，以便在godoc中容易查找。

注释应该以一个句点`.`结束。

### 声明slice

声明空的slice应该使用下面的格式:

```go
var t []string                  
```

而不是这种格式:
```go

t := []string{}                 


前者声明了一个`nil` slice而后者是一个长度为0的非nil的slice。

### 关于字符串大小写

错误字符串不应该大写。\
应该写成：

```go
fmt.Errorf("failed to write data")                                  
```

而不是写成：

```go
fmt.Errorf("Failed to write data")                                  
```

这是因为这些字符串可能和其它字符串相连接，组合后的字符串如果中间有大写字母开头的单词很突兀，除非这些首字母大写单词是固定使用的单词。

缩写词必须保持一致，比如都大写`URL`或者小写`url`。比如`HTTP`、`ID`等。
例如`sendOAuth`或者`oauthSend`。

maxLength not MaxLength or MAX\_LENGTH.

常量一般声明为`MaxLength`,而不是以下划线分隔`MAX_LENGTH`或者`MAXLENGTH`。

也就是Go语言一般使用`MixedCaps`或者`mixedCaps`命名的方式区分包含多个单词的名称。

### 处理error而不是panic或者忽略

为了编写强壮的代码，不用使用`_`忽略错误，而是要处理每一个错误，尽管代码写起来可能有些繁琐。

尽量不要使用panic。

### 尽量减少代码缩进

### 一些名称

有些单词可能有多种写法，在项目中应该保持一致，比如Golang采用的写法:

```go
  // marshaling
  // unmarshaling
  // canceling
  // cancelation              
```

而不是
```go
// marshalling
// unmarshalling
// cancelling
// cancellation          
```

包名应该用单数的形式，比如`util`、`model`,而不是`utils`、`models`。

Receiver 的名称应该缩写，一般使用一个或者两个字符作为Receiver的名称，如

```go
func (f foo) method() {...}
``` 
如果方法中没有使用receiver,还可以省略receiver
name,这样更清晰的表明方法中没有使用它:
```go
func (foo) method() {...}
```     


### package级的Error变量 {#package级的Error变量}

通常会把自定义的Error放在package级别中，统一进行维护:
```go
var (  
	ErrCacheMiss = errors.New("memcache: cache miss")   
	ErrCASConflict = errors.New("memcache: compare-and-swap conflict")  
	ErrNotStored = errors.New("memcache: item not stored")
	ErrServerError = errors.New("memcache: server error")   
	ErrNoStats = errors.New("memcache: no statistics available") 
	ErrMalformedKey =errors.New("malformed: key is too long or contains invalid characters")
	ErrNoServers = errors.New("memcache: no servers configured or available"))                               
```

并且变量以`Err`开头。

### 空字符串检查

不要使用下面的方式检查空字符串:
```go
if len(s) == 0 { ...}     
```     
而是使用下面的方式
```go
if s == "" { ...} 
```              


下面的方法更是语法不对：

```go
if s == nil| s == "" { ...}   
```

### 非空slice检查

不要使用下面的方式检查空的slice:
```go
if s != nil && len(s) > 0 {...}                                 
```

直接比较长度即可：

```go
if len(s) > 0 {    ...}         
```

同样的道理也适用 `map`和`channel`。

### 省略不必要的变量

比如
```go
var whitespaceRegex, _ = regexp.Compile("\\s+")                     
```

可以简写为
```go
var whitespaceRegex = regexp.MustCompile(`\s+`)                     
```

有时候你看到的一些第三方的类提供了类似的方法:

```go
func Foo(...) (...,error)
func MustFoo(...) (...)                    
```

MustFoo一般提供了一个不带error返回的类型。

### 直接使用bool值

对于bool类型的变量`var b bool`,直接使用它作为判断条件，而不是使用它和true/false进行比较
```go
if b {    ...}
if !b {    ...}   
```
而不是
```go
if b == true {    ...}
if b == false {    ...}                       
```

### byte/string slice相等性比较 {#byte/string_slice相等性比较}

不要使用

```go
var s1 []byte   
var s2 []byte
...
bytes.Compare(s1, s2) == 0
bytes.Compare(s1, s2) != 0              
```
而是:
```go
var s1 []byte   
var s2 []byte
...
bytes.Equal(s1, s2) == 0bytes.
Equal(s1, s2) != 0                  
```

### 检查是否包含子字符串

不要使用
`strings.IndexRune(s1, 'x') > -1`及其类似的方法`IndexAny`、`Index`检查字符串包含，\
而是使用`strings.ContainsRune`、`strings.ContainsAny`、`strings.Contains`来检查。

### 使用类型转换而不是struct字面值

对于两个类型:
```go
type t1 struct {    a int b int}
type t2 struct {    a int b int}    
```

可以使用类型转换将类型t1的变量转换成类型t2的变量，而不是像下面的代码进行转换
```go
v1 := t1{1, 2}
_ = t2{v1.a, v1.b}
```

应该使用类型转换，因为这两个struct底层的数据结构是一致的。
```go
_ = t2(v1)                      
```

### 复制slice

不要使用下面的复制slice的方式:

```go
var b1, b2 []byte
for i, v := range b1 {
    b2[i] = v
}
for i := range b1 {
    b2[i] = b1[i]
}            


而是使用内建的`copy`函数：
```go
copy(b2, b1)                    
```

### 不要在for中使用多此一举的true

不要这样:
```go
for true {}                     
```

而是要这样:
```go
for {}                          
```

### 尽量缩短if

下面的代码:
```go
x := true
if x {   
	return true
}
return false                          
```

可以用`return x`代替。

同样下面的代码也可以使用`return err`代替：
```go
func fn1() error { 
 var err error  
 if err != nil { 
 	return err  
 }  
 return nil
 }

func fn()   bool{   
	if b {       
 	  ...         
  	 return true     
	} else {
  	  return false    
	}
}                      
```

也就是减少if的分支／缩进。


不要这样:

```go
var a, b []int
for _, v := range a {  
	b = append(b, v)
}             
```

而是要这样
```go
var a, b []int
b = append(b, a...)                                   
```

### 简化range
```go
var m map[string]int
for _ = range m { }
for _, _ = range m {}        
```

可以简化为
```go
for range m {}                  
```

对`slice`和`channel`也适用。

### 正则表达式中使用`raw`字符串避免转义字符

在使用正则表达式时，不要:
```go
regexp.MustCompile("\\.") 
regexp.Compile("\\.")                     
```

而是直接使用raw字符串，可以避免大量的`\`出现：

```go
regexp.MustCompile(`\.`) 
regexp.Compile(`\.`)                       
```

### 简化只包含单个case的select
```go
select {   case <-ch:}          
```
直接写成`<-ch`即可。send也一样。
```go
for {    
	select {   
		case x := <-ch:         
		_ = x
	}                 
}
```
直接改成 `for-range`即可。

这种简化只适用包含单个case的情况。

### slice的索引

有时可以忽略slice的第一个索引或者第二个索引：

```go
var s []int
_ = s[:len(s)]
_ = s[0:len(s)]                            
```
可以写成`_ =s[:]`

### 使用time.Since {#使用time-Since}

下面的代码经常会用到：
```go
_ = time.Now().Sub(t1)          
```

可以简写为:
```go
_ = time.Since(t1)              
```

### 使用strings.TrimPrefix／strings.TrimSuffix 掐头去尾 {#使用strings-TrimPrefix／strings-TrimSuffix_掐头去尾}

不要自己判断字符串是否以XXX开头或者结尾，然后自己再去掉XXX,而是使用现成的`strings.TrimPrefix／strings.TrimSuffix`。
```go
var s1 = "a string value"
var s2 = "a "
var s3 string
if strings.HasPrefix(s1, s2) {    
	s3 = s1[len(s2):]
} 
```
可以简化为
```go
var s1 = "a string value"
var s2 = "a "
var s3 = strings.TrimPrefix(s1, s2)                               
```

### 使用工具检查你的代码

以上的很多优化规则都可以通过工具检查，下面列出了一些有用的工具：

1.  [go fmt](https://golang.org/pkg/fmt/)
2.  [go vet](https://golang.org/cmd/vet/)
3.  [gosimple](https://github.com/dominikh/go-tools/blob/master/cmd/gosimple)
4.  [keyify](https://github.com/dominikh/go-tools/blob/master/cmd/keyify)
5.  [staticcheck](https://github.com/dominikh/go-tools/blob/master/cmd/staticcheck)
6.  [unused](https://github.com/dominikh/go-tools/blob/master/cmd/unused)
7.  [golint](https://github.com/golang/lint)
8.  [misspell](https://github.com/client9/misspell)
9.  [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports)
10. [errcheck](https://github.com/kisielk/errcheck)

### 参考文档

1.  [https://golang.org/doc/effective\_go.html](https://golang.org/doc/effective_go.html)
2.  [https://github.com/golang/go/wiki/CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)
3.  [https://dmitri.shuralyov.com/idiomatic-go](https://dmitri.shuralyov.com/idiomatic-go)
4.  [https://talks.golang.org/2014/readability.slide](https://talks.golang.org/2014/readability.slide)
5.  [https://github.com/dominikh/go-tools/tree/master/simple](https://github.com/dominikh/go-tools/tree/master/simple)
6.  [https://github.com/dominikh/go-tools/tree/master/cmd/structlayout-optimize](https://github.com/dominikh/go-tools/tree/master/cmd/structlayout-optimize)
7.  [https://go-zh.org](https://go-zh.org)
8.  [https://docs.google.com/presentation/d/1OT-dMNbiwOPeaivQOldok2hUUNMTvSC0GJ67JohLt5U/pub?start=false&loop=false&delayms=3000&slide=id.g18b1f95882\_1\_135](https://docs.google.com/presentation/d/1OT-dMNbiwOPeaivQOldok2hUUNMTvSC0GJ67JohLt5U/pub?start=false&loop=false&delayms=3000&slide=id.g18b1f95882_1_135)
9.  [https://github.com/d-smith/go-training/blob/master/idiomatic-go.md](https://github.com/d-smith/go-training/blob/master/idiomatic-go.md)


