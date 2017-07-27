---
date: 2013-04-04
title: 老虞学Golang-数组和切片
slug: ysqi-golang-array-slice
topics:
- 编程语言与开发
tags:
- Golang
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100014
---

### 数组 Arrays

数组是内置(build-in)类型,是一组同类型数据的集合，它是值类型，通过从0开始的下标索引访问元素值。在初始化后长度是固定的，无法修改其长度。当作为方法的入参传入时将复制一份数组而不是引用同一指针。数组的长度也是其类型的一部分，通过内置函数len(array)获取其长度。

#### 初始化

数组的初始化有多种形式,[查看示例代码](arraysInitDemo) , [在线运行示例代码](arraysInitDemo_play)

+ `[5] int {1,2,3,4,5} `
   <br/>长度为5的数组，其元素值依次为：1，2，3，4，5
+ `[5] int {1,2}`
   <br/>长度为5的数组，其元素值依次为：1，2，0，0，0 。在初始化时没有指定初值的元素将会赋值为其元素类型int的默认值0,string的默认值是""
+ `[...] int {1,2,3,4,5}`
   <br/>长度为5的数组，其长度是根据初始化时指定的元素个数决定的
+ `[5] int { 2:1,3:2,4:3}`
   <br/>长度为5的数组，key:value,其元素值依次为：0，0，1，2，3。在初始化时指定了2，3，4索引中对应的值：1，2，3
+ `[...] int {2:1,4:3}`
   <br/>长度为5的数组，起元素值依次为：0，0，1，0，3。由于指定了最大索引4对应的值3，根据初始化的元素个数确定其长度为5



#### 赋值与使用

数组通过下标访问元素，可修改其元素值
```Go
arr :=[...] int {1,2,3,4,5}
arr[4]=arr[1]+len(arr)      //arr[4]=2+5
```

通过for遍历数组元素,[查看示例代码](arraysForDemo),[在线运行示例代码](arraysForDemo_play)
```Go
arr := [5]int{5, 4, 3}

for index, value := range arr {
	fmt.Printf("arr[%d]=%d \n", index, value)
}

for index := 0; index < len(arr); index++ {
	fmt.Printf("arr[%d]=%d \n", index, arr[index])
}
```

数组是值类型，将一个数组赋值给另一个数组时将复制一份新的元素,[查看示例代码](arraysValueDemo),[在线运行示例代码](arraysValueDemo_play)
```Go
	arr2 := [5]int{1, 2}
	arr5 := arr2
	arr5[0] = 5
	arr2[4] = 2
	fmt.Printf(" arr5= %d \n arr2=%d \n arr5[0]==arr2[0]= %s \n", arr5, arr2, arr5[0] == arr2[0])

	OutPut:
	 arr5=[5 2 0 0 0]
     arr2=[1 2 0 0 2]
     arr5[0]==arr2[0]= false
```

### 切片 Slices

数组的长度不可改变，在特定场景中这样的集合就不太适用，Go中提供了一种灵活，功能强悍的内置类型**Slices**切片,与数组相比切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大。切片中有两个概念：一是**len长度**，二是**cap容量**，长度是指已经被赋过值的最大下标+1，可通过内置函数len()获得。容量是指切片目前可容纳的最多元素个数，可通过内置函数cap()获得。切片是引用类型，因此在当传递切片时将引用同一指针，修改值将会影响其他的对象。

#### 初始化

切片可以通过数组来初始化，也可以通过内置函数make()初始化 .初始化时len=cap,在追加元素时如果容量cap不足时将按len的**2**倍扩容
[查看示例代码](slicesInitDemo)，[在线运行示例代码](slicesInitDemo_play)

+  `s :=[] int {1,2,3 }`
   <br/>直接初始化切片，**[]**表示是切片类型，**{1,2,3}**初始化值依次是1,2,3.其cap=len=3
+  `s := arr[:] `
   <br/>初始化切片s,是数组arr的引用
+  `s := arr[startIndex:endIndex] `
   <br/>将arr中从下标startIndex到endIndex-1 下的元素创建为一个新的切片
+  `s := arr[startIndex:] `
   <br/>缺省endIndex时将表示一直到arr的最后一个元素
+  `s := arr[:endIndex]	`
   <br/>缺省startIndex时将表示从arr的第一个元素开始
+  `s1 := s[startIndex:endIndex] `
   <br/>通过切片s初始化切片s1
+  `s :=make([]int,len,cap) `
   <br/>通过内置函数make()初始化切片s,[]int 标识为其元素类型为int的切片


#### 赋值与使用

切片是引用类型，在使用时需要注意其操作。[查看示例代码](slicesValueDemo) ，[在线运行示例代码](slicesValueDemo_play)
切片可以通过内置函数append(slice []Type,elems ...Type)追加元素，elems可以是一排type类型的数据，也可以是slice,因为追加的一个一个的元素，因此如果将一个slice追加到另一个slice中需要带上"**...**"，这样才能表示是将slice中的元素依次追加到另一个slice中。另外在通过下标访问元素时下标不能超过len大小，如同数组的下标不能超出len范围一样。

+ `s :=append(s,1,2,3,4)`
+ `s :=append(s,s1...)`



[arraysInitDemo_play]:http://play.golang.org/p/SHJBJ8hAw1
[arraysInitDemo]:https://github.com/devYu/GoLangStudy/blob/master/codeDemo/arrays/arraysInitDemo.go
[arraysValueDemo_play]:http://play.golang.org/p/3ejYrjlfnm
[arraysValueDemo]:https://github.com/devYu/GoLangStudy/blob/master/codeDemo/arrays/arraysValueDemo.go
[arraysForDemo_play]:http://play.golang.org/p/XyypK-D7Bk
[arraysForDemo]:https://github.com/devYu/GoLangStudy/blob/master/codeDemo/arrays/arraysForDemo.go

[slicesInitDemo_play]:http://play.golang.org/p/7srUx-HM0p
[slicesInitDemo]:https://github.com/devYu/GoLangStudy/blob/master/codeDemo/arrays/slicesInitDemo.go
[slicesValueDemo_play]:http://play.golang.org/p/sQgYIurZoy
[slicesValueDemo]:https://github.com/devYu/GoLangStudy/blob/master/codeDemo/arrays/slicesValueDemo.go
