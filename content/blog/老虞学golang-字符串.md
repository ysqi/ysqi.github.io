---
date: 2013-04-06
title: 老虞学golang-字符串
slug: ysqi-golang-string
topics:
- 开发
tags:
- Go
- 笔记
- 教程
books:
- 老虞学GoLang
---

> **在所有编程语言中都涉及到大量的字符串操作，可见熟悉对字符串的操作是何等重要。
>Go中的字符串和C#中的一样，字符串内容在初始化后不可修改。
>需要注意的是在Go中字符串是有UTF-8编码的，请注意保存文件时将文件编码格式改成UTF-8(特别是在windows下)。**

## 初始化
```golang
    var str string //声明一个字符串
    str = "laoYu"  //赋值
    ch :=str[0]    //获取第一个字符
    len :=len(str) //字符串的长度,len是内置函数 ,len=5
```

## 字符串操作

>**编码过程中避免不了中文字符，那我们该如何提取一个中文呢？首先我们要知道string[index]获取的是字符byte,就无法像C#中`"老虞"[0]`来取到‘老’，在Go中需要将字符串转换成rune数组，runne数组中就可以通过数组下标获取一个汉字所标识的Unicode码，再将Unicode码按创建成字符串即可。**

[查看示例代码][stringDemo]
```golang
    str :="laoYu老虞"

    for  i:=0;i<len(str);i++ {
             fmt.Println(str[i])
    }

    for  i,s :=  range str {
            fmt.Println(i,"Unicode(",s,") string=",string(s))
    }

    r := []rune(str)
    fmt.Println("rune=",r)
    for i:=0;i<len(r) ; i++ {
           fmt.Println("r[",i,"]=",r[i],"string=",string(r[i]))
    }

    Outut：
    108
    97
    111
  	89
  	117
  	232
  	128
  	129
  	232
  	153
  	158
  	0 Unicode( 108 ) string= l
  	1 Unicode( 97 ) string= a
  	2 Unicode( 111 ) string= o
  	3 Unicode( 89 ) string= Y
  	4 Unicode( 117 ) string= u
  	5 Unicode( 32769 ) string= 老
  	8 Unicode( 34398 ) string= 虞
  	rune= [108 97 111 89 117 32769 34398]
  	r[ 0 ]= 108 string= l
  	r[ 1 ]= 97 string= a
  	r[ 2 ]= 111 string= o
  	r[ 3 ]= 89 string= Y
  	r[ 4 ]= 117 string= u
  	r[ 5 ]= 32769 string= 老
  	r[ 6 ]= 34398 string= 虞
```
### 对字符串的操作非常重要，来了解下strings包中提供了哪些函数


**获取总字节数** `func Len(v type) int `


>len函数是Go中内置函数，不引入strings包即可使用。len(string)返回的是字符串的字节数。len函数所支持的入参类型如下：

> + len(Array)  数组的元素个数
> + len(*Array) 数组指针中的元素个数,如果入参为nil则返回0
> + len(Slice)  数组切片中元素个数,如果入参为nil则返回0
> + len(map)    字典中元素个数,如果入参为nil则返回0
> + len(Channel) Channel buffer队列中元素个数

[查看示例代码][lenDemo]
```golang
    str :="laoYu老虞"
    str2 :="laoYu"
    fmt.Println("len(",str,")=",len(str))      //len=11=5+6,一个汉字在UTF-8>中占3个字节
    fmt.Println("len(",str2,")=",len(str2))    //len=5
    fmt.Println("str[0]=",str[0])              //l

    str :="str"
    arr :=[5]int{1,2,3}
    slice :=make([]int,5)

    m :=make(map[int] string)
    m[2]="len"

    ch :=make(chan int)

    fmt.Println("len(string)=",len(str))   //3
    fmt.Println("len(array)=",len(arr))     //5invalid argument user (type *UserInfo) for len

    fmt.Println("len(slice)=",len(slice))   //5
    fmt.Println("len(map)=",len(m))         //1
    fmt.Println("len(chat)=",len(ch))       //0

    //user :=&UserInfo{id:1,name:"laoYu"}
    //interger :=2
    //fmt.Println("len(my struct)=",len(user))//invalid argument user (type *UserInfo) for len
    //fmt.Println("len(interger)=",len(interger))

    var str2 string
    var arr2  [5]int
    var slice2  []int
    var  m2 map[int] string
    var  ch2 chan int

    fmt.Println("len(string)=",len(str2))    //0
    fmt.Println("len(array)=",len(arr2))     //5
    fmt.Println("len(slice)=",len(slice2))   //0
    fmt.Println("len(map)=",len(m2))         //0
    fmt.Println("len(chat)=",len(ch2))       //0
```
**字符串中是否包含某字符串** `func Contains(s, substr string) bool `

> 确定是否包含某字符串，这是区分大小写的。实际上内部是通过Index(s,sub string) int 实现的。如果索引!=-1则表示包含该字符串。空字符串""在任何字符串中均存在。

**源代码**
```golang
    // Contains returns true if substr is within s.
    func Contains(s, substr string) bool {
         return Index(s, substr) != -1
    }
```
**示例，使用请需导入包:' import "strings" ' **

[查看示例代码][contiansDemo]
```golang
    str :="laoYuStudyGotrue是否包含某字符串"
    fmt.Println(strings.Contains(str,"go"))         //false
    fmt.Println(strings.Contains(str,"Go"))         //true
    fmt.Println(strings.Contains(str,"laoyu"))      //false
    fmt.Println(strings.Contains(str,"是"))         //true
	fmt.Println(strings.Contains(str,"")) 			//true
```

> 在实际工作中常需要在不区分大小写的情况下确认是否包含某字符串,(我们应该减少这种情况，以免每次验证时都需要进行一次大小写转换)。
> 这里我局部修改源代码提供一个验证字符串中是否包含某字符串的函数,当然你也可以直接使用`strings.Contains(strings.ToLower(s),strings.ToLower(substr))`
```golang
    str := "laoYuStudyGotrue是否包含某字符串"
    fmt.Println(Contains(str, "go", true))  //true
    fmt.Println(Contains(str,"go",false))   //false
```

 ```golang
		//在字符串s中是否包含字符串substr,ignoreCase表示是否忽略大小写
		func Contains(s string, substr string, ignoreCase bool) bool {
				return Index(s, substr, ignoreCase) != -1

		}

		//字符串subst在字符串s中的索引位置,ignoreCase表示是否忽略大小写
		func Index(s string, sep string, ignoreCase bool) int {

				n := len(sep)
				if n == 0 {
				        return 0
				}

				//to Lower
				if ignoreCase == true {
				        s = strings.ToLower(s)
				        sep = strings.ToLower(sep)
				}

				c := sep[0]
				if n == 1 {
				        // special case worth making fast
				        for i := 0; i < len(s); i++ {
				                if s[i] == c {
				                        return i
				                }
				        }
				        return -1
				}
				// n > 1
				for i := 0; i+n <= len(s); i++ {
				        if s[i] == c && s[i:i+n] == sep {
				                return i
				        }
				}
				return -1
		}
```

** 获取字符串sep在字符串s中出现的次数 ** `Count(s,sep string)`
> 注意：如果sep=""，则无论s为何字符串都会返回 len(s)+1

[查看示例代码][countDemo]
```golang
    fmt.Println(strings.Count("laoYuStudyGo", "o"))                 //2
    fmt.Println(strings.Count("laoYuStudyGo", "O"))                 //0
    fmt.Println(strings.Count("laoYuStudyGo", ""))                  //13=12+1
    fmt.Println(strings.Count("laoYuStudyGo老虞学习Go语言", "虞"))  //1
    fmt.Println(strings.Count("laoYuStudyGo老虞学习Go语言", "Go"))  //2
    fmt.Println(strings.Count("laoYuStudyGo老虞学习Go语言", "老虞"))//1
    fmt.Println(strings.Count("", ""))                             //1=0+1
    fmt.Println(strings.Count("aaaaaaaa","aa"))                     //4
    fmt.Println(strings.Count("laoYuStudyGo_n","\n"))               //0
```
** 使用（多个）空格分割字符串 Fields(s string) ,返回分割后的数组 **
> 将字符串分割成数组，其分割符为空格。

[查看示例代码][fieldsDemo]
```golang
    fmt.Println(strings.Fields("lao Yu Study Go ")) //OutPut: [lao Yu Study Go]
    fmt.Println(strings.Fields("   Go    "))        //[Go]
    fmt.Println(strings.Fields(""))                 //[]
    fmt.Println(strings.Fields(" \n go"))           //[go]
```
** 其实其内部实现调用的是FieldsFunc(s,unicode.IsSpace),我们也可以自定义分割方式 **
```golang
    canSplit := func (c rune)  bool { return c=='#'}
    fmt.Println(strings.FieldsFunc("lao###Yu#Study####Go#G ",canSplit)) //[lao Yu Study Go G<space>]
```
** 检查字符串是否已某字符串开头 ** HasPrefix(s,prefix string) bool

** 如果想查看更多关于strings包下的字符串操作函数，请查看 **

 + [官方strings包地址][stringsPkgUrl]
 + [对应的函数翻译地址][stringsPkgUrl_zh-cn]

[stringDemo]: https://github.com/devYu/GoLangStudy/blob/master/codeDemo/strings/stringDemo.go
[lenDemo]: https://github.com/devYu/GoLangStudy/blob/master/codeDemo/strings/lenDemo.go
[contiansDemo]: https://github.com/devYu/GoLangStudy/blob/master/codeDemo/strings/containsDemo.go
[countDemo]: https://github.com/devYu/GoLangStudy/blob/master/codeDemo/strings/countDemo.go
[fieldsDemo]: https://github.com/devYu/GoLangStudy/blob/master/codeDemo/strings/fieldsDemo.go
[stringsPkgUrl]: https://golang.org/pkg/strings
[stringsPkgUrl_zh-cn]: https://github.com/astaxie/gopkg/tree/master/strings
