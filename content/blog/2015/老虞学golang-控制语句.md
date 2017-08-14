---
title: 老虞学Golang-控制语句
slug: ysqi-golang-if-for-select-switch
date: 2013-04-08T23:31:47
topics:
- 编程语言与开发
tags:
- Golang
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100017
---

Go中的控制语句较精简，仅有if、for、select和switch。但使用时均比较灵活


if
==

在Go中条件语句*if*中如果条件部分的计算结果为**true**时将执行语句块，否则则执行else语句块(如果存在else时)，此逻辑和其他语言中的if一样，但是在Go中还是有一些不同之处。

+ if条件表达式不能使用花括号**()**包含
+ if语句代码段必须使用**{}**，并且左括号必须和if在同一行
+ if条件表达式的前面可以包含初始化语句，支持平行赋值，但不支持多个赋值语句

赋值+条件判断
```Go
	if a, b := 21, 3; a > b {
		fmt.Println("a>b ? true")
	}
```
在if条件表达式前面声明的的变量只能在if-else语句块中使用。
```Go
    if a, b := 21, 31; a > b {
		fmt.Println("a>b ? true")
	}else {
		fmt.Println(a,b) //Ok
	}
	fmt.Println(a,b)    //error: undefined a ,undefined b
```

还需要注意的是如果在`if-else` 中包含return 时，编译器无法解析出else中的retrun,导致方法缺少return ,目前1.1版本已支持该方式。
```Go
	func getName(id int) string {
		if id == 1 {
			return "YourName"
		}else {
			return "MyName"  
		}
        //panic("")
	}
```

此代码编译不通过，错误信息：`function ends without a return statement`，这是在设计Go时故意这样的,也可以说是一个Bug（可参见:[https://code.google.com/p/go/issues/detail?id=65](https://code.google.com/p/go/issues/detail?id=65)），这是一种编码风格，即在if语句块中去做return处理,而else中不处理，而是继续执行if-else后面的代码，这样能减少一个代码缩进，不需要在了解代码时去记住else语句块的处理。当然如果想必须这样写,也可以进行特殊处理，在函数的末行添加语句**panic("")**

 在if中可以包含初始化语句，这是非常实用的。例如在文件处理，取字典项时需要判断是否执行操作成功，只有在成功时才能继续处理，这样就可以通过if-else处理。
```Go
	if err := file.Chmod(0664); err != nil {
	    log.Print(err)
	    return err
	}

	if v,err :=myMap[key];err != nil {
		log.Print(err)
		return err
	}else {
		//do something with v
	}
```

for
===

在Go中其他循环遍历的控制语句，唯有for。而for同样也是比较灵活的，for有三种形式。

+ 普通用法         `for init; condition;post {}`
+ while           `for condition {}`
+ 死循环           `for {}`

由于Go没有逗号表达式，而++和--是语句而不是表达式，如果想在for中执行多个变量,需要使用平行赋值
```Go
	for i, j := 1, 10; i < j; i,j=i+1,j+1 {  //死循环
		fmt.Println(i)
	}
```
而不能写成
```Go
	for i, j := 1, 10; i < j; i++,j++ {
		fmt.Println(i)
	}
```

for的condition在每执行一次循环体时便会执行一次，因此在实际开发过程中需要注意不要让condition中计算简单而不是复杂。

	for i,j :=0,len(str); i<j ; i++ {
		fmt.Println(str[i])
	}

而不要写成(这仅是一个演示而已)

	for i=0; i< len(str); i++ {
		fmt.Println(str[i])
	}

另外for是遍历string,array,slice,map,chanel的方式,而使用保留字rang则能灵活的处理。rang是迭代器，能根据不同的内容，返回不同的东西。

+ `for index,char  := range string {}`
+ `for index,value := range array  {}`
+ `for index,value := range slice  {}`
+ `for key,value   := range map    {}`

需要注意的是for+rang遍历string时得到的是字节索引位置和UTF-8格式rune类型数据(int32)。
```Go
	for pos, value := range "Go在中国" {
		fmt.Printf("character '%c' type is %T value is %v, and start at byte position %d \n", value,value,value, pos)
	 	str :=string(value)  //convert rune to string
		fmt.Printf("string(%v)=>%s \n",value,str)
	}
    ---------OutPut------------ 一个汉字占三个字节
	character 'G' type is int32 value is 71, and start at byte position 0
	string(71)=>G
	character 'o' type is int32 value is 111, and start at byte position 1
	string(111)=>o
	character '在' type is int32 value is 22312, and start at byte position 2
	string(22312)=>在
	character '中' type is int32 value is 20013, and start at byte position 5
	string(20013)=>中
	character '国' type is int32 value is 22269, and start at byte position 8
	string(22269)=>国
```

###break和continue
 另外在循环中使用break和continue，break用于退出循环，continue用于终止本次循环体的执行继续执行下一个循环。
```Go
	sum := 0
	for {
		if sum > 10 {
			break
		}
		sum += 2
	}
	fmt.Println(sum) // sum=12
```
break也可退出指定的循环体
```Go
		sum := 0
	myforLable:
		for {
			for i := 0; i < 10; i++ {
				if sum > 200 {
					break myforLable //将退出循环体for{}
				}
				sum += i
			}
		}
		fmt.Println(sum)
```

switch
===

switch是最灵活的一种控制语句,表达式可以不是常量或者字符串，也可以沒有表达式，如果没有表达式则如同if-else-else。

一般用法:和其他语言的Switch基本一样，不同的不需要在每个Case中添加Break,而是隐藏了Break,当然你可以显示加入break
```Go
	switch ch {
		case '0':
			cl = "Int"
		case '1':
			cl = "Int"
		case 'A':
			cl = "ABC"
			break //可以添加
		case 'a':
			cl = "ABC"
		default:
			cl = "Other Char"
	 }
```
 此段代码可以可写成(fallthrough表示继续执行下面的Case而不是退出Switch)

	switch ch {
	case '0':
		fallthrough   //必须是最后一个语句
	case '1':
		cl = "Int"
	case 'A':
	case 'a':
	    fallthrough
		cl = "ABC"    //error
	default:
		cl = "Other Char"
	}


如果多个匹配结果所对应的代码段一样，则可以在一个case中并列出所有的匹配项
```Go
	switch ch {
		case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
			cl = "Int"
		case 'A', 'B', 'C', 'D',  'a', 'b', 'c', 'd', 'e':
			cl = "ABC"
		default:
			cl = "Other Char"
	}
```
同样switch可以没有表达式，在 Case 中使用布尔表达式，这样形如 if-else
```Go
	switch {
		case '0' <= ch && ch <= '9':
			cl = "Int"
		case ('a' <= ch && ch <= 'z') || ('A' <= ch && ch <= 'Z'):
			cl = "ABC"
		default:
			cl = "Other Char"
	}
```

下面是Switch写的一个示例（无实际意义）：
```Go
	func Compare(a, b interface{}) (int, error) {
		aT := reflect.TypeOf(a)
		bT := reflect.TypeOf(b)
		if aT != bT {
			return -2, errors.New("进行比较的数据类型不一致")
		}

		switch av := a.(type) {

		default:
			return -2, errors.New("该类型数据比较没有实现")
		case string:
			switch bv := b.(type) {
			case string:
				return ByteCompare(av, bv), nil
			}
		case int:
			switch bv := b.(type) {
			case int:
				return NumCompare(av, bv), nil
			}

		}

		return -2, nil
    }
```

select
===

  还有另一个控制语句select，在讨论chan 时再来学习

本笔记中所写代码存储位置：

+ [defer.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/defer.go)
+ [defineFunctionType.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/defineFunctionType.go)
+ [function.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/function.go)


[上一篇(数组和切片)](https://github.com/devYu/GoLangStudy/blob/master/myNotes/arrayAndSlice.md)

[下一篇（函数上）](https://github.com/devYu/GoLangStudy/blob/master/myNotes/function.md)
