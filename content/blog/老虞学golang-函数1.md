---
date: 2013-04-07
title: 老虞学Golang-函数上
slug: ysqi-golang-function-1
topics:
- 开发
tags:
- Go
- 笔记
- 教程
books:
- 老虞学GoLang
---
不可或缺的函数，在Go中定义函数的方式如下：
```golang
func (p myType ) funcName ( a, b int , c string ) ( r , s int ) {
	return
}
```
通过函数定义，我们可以看到Go中函数和其他语言中的共性和特性

### 共性
+ 关键字——func
+ 方法名——funcName
+ 入参——— a,b int,b string
+ 返回值—— r,s int
+ 函数体—— {}


### 特性
  Go中函数的特性是非常酷的，给我们带来不一样的编程体验。

#### 为特定类型定义函数，即为类型对象定义方法

 在Go中通过给函数标明所属类型，来给该类型定义方法，上面的 `p myType` 即表示给myType声明了一个方法， `p myType` 不是必须的。如果没有，则纯粹是一个函数，通过包名称访问。packageName.funcationName

如：
```golang
	//定义新的类型double，主要目的是给float64类型扩充方法
	type double float64

	//判断a是否等于b
	func (a double) IsEqual(b double) bool {
		var r = a - b
		if r == 0.0 {
			return true
		} else if r < 0.0 {
			return r > -0.0001
		}
		return r < 0.0001
	}

	//判断a是否等于b
	func IsEqual(a, b float64) bool {
		var r = a - b
		if r == 0.0 {
			return true
		} else if r < 0.0 {
			return r > -0.0001
		}
		return r < 0.0001
	}

	func main() {
		var a double = 1.999999
		var b double = 1.9999998
		fmt.Println(a.IsEqual(b))
		fmt.Println(a.IsEqual(3))
		fmt.Println( IsEqual( (float64)(a), (float64)(b) ) )

	D}
```
上述示例为 float64 基本类型扩充了方法IsEqual，该方法主要是解决精度问题。
其方法调用方式为：  `a.IsEqual(double)` ，如果不扩充方法，我们只能使用函数`IsEqual(a, b float64) `

#### 入参中，如果连续的参数类型一致，则可以省略连续多个参数的类型，只保留最后一个类型声明。

 如  `func IsEqual(a, b float64) bool ` 这个方法就只保留了一个类型声明,此时入参a和b均是float64数据类型。
这样也是可以的： `func IsEqual(a, b float64, accuracy int) bool `

#### 变参：入参支持变参,即可接受不确定数量的同一类型的参数

如 `func Sum(args ...int)` 参数args是的slice，其元素类型为int 。经常使用的fmt.Printf就是一个接受任意个数参数的函数 `fmt.Printf(format string, args ...interface{})`

#### 支持多返回值
 前面我们定义函数时返回值有两个r,s 。这是非常有用的，我在写C#代码时，常常为了从已有函数中获得更多的信息，需要修改函数签名，使用out ,ref 等方式去获得更多返回结果。而现在使用Go时则很简单，直接在返回值后面添加返回参数即可。

如,在C#中一个字符串转换为int类型时逻辑代码
```golang
    int v=0;
	if ( int.TryPase("123456",out v) )
	{
		//code
	}
```
而在Go中，则可以这样实现,逻辑精简而明确
```golang
	if v,isOk :=int.TryPase("123456") ; isOk {
		//code
	}
```
同时在Go中很多函数充分利用了多返回值

+ func (file *File) Write(b []byte) (n int, err error)
+ func Sincos(x float64) (sin, cos float64)

那么如果我只需要某一个返回值，而不关心其他返回值的话，我该如何办呢？ 这时可以简单的使用符号下划线”_“ 来忽略不关心的返回值。如：
```golang
	_, cos = math.Sincos(3.1415) //只需要cos计算的值
```
#### 命名返回值

  前面我们说了函数可以有多个返回值，这里我还要说的是，在函数定义时可以给所有的返回值分别命名，这样就能在函数中任意位置给不同返回值复制，而不需要在return语句中才指定返回值。同时也能增强可读性，也提高godoc所生成文档的可读性

如果不支持命名返回值，我可能会是这样做的
```golang
	func ReadFull(r Reader, buf []byte) (int, error) {
		var n int
		var err error

	    for len(buf) > 0  {
	        var nr int
	        nr, err = r.Read(buf)
	        n += nr
			if err !=nil {
				return n,err
			}
	        buf = buf[nr:]
	    }
	    return n,err
	}
```
但支持给返回值命名后，实际上就是省略了变量的声明，return时无需写成`return n,err`
而是将直接将值返回
```golang
	func ReadFull(r Reader, buf []byte) (n int, err error) {
	    for len(buf) > 0 && err == nil {
	        var nr int
	        nr, err = r.Read(buf)
	        n += nr
	        buf = buf[nr:]
	    }
	    return
	}
```
#### 函数也是“值”

   和Go中其他东西一样，函数也是值，这样就可以声明一个函数类型的变量，将函数作为参数传递。

声明函数为值的变量(匿名函数:可赋值个变量，也可直接执行)
```golang
	//赋值
	fc := func(msg string) {
		fmt.Println("you say :", msg)
	}
	fmt.Printf("%T \n", fc)
	fc("hello,my love")
	//直接执行
	func(msg string) {
		fmt.Println("say :", msg)
	}("I love to code")
```
输出结果如下，这里表明fc 的类型为：func(string)
```golang
    func(string)
	you say : hello,my love
	say : I love to code
```
将函数作为入参（回调函数），能带来便利。如日志处理，为了统一处理，将信息均通过指定函数去记录日志，且是否记录日志还有开关
```golang
	func Log(title string, getMsg func() string) {
		//如果开启日志记录,则记录日志
		if true {
			fmt.Println(title, ":", getMsg())
		}
	}
    //---------调用--------------
	count := 0
	msg := func() string {
		count++
		return "您没有即使提醒我,已触犯法律"
	}
	Log("error", msg)
	Log("warring", msg)
	Log("info", msg)
	fmt.Println(count)
```
这里输出结果如下，count 也发生了变化
```golang
	error : 您没有即使提醒我,已触犯法律
	warring : 您没有即使提醒我,已触犯法律
	info : 您没有即使提醒我,已触犯法律
```
#### 函数也是“类型”
你有没有注意到上面示例中的 `fc := func(msg string)...` ，既然匿名函数可以赋值给一个变量，同时我们经常这样给int赋值 ` value := 2` ,是否我们可以声明func(string)
类型	呢，当然是可以的。
```golang
	//一个记录日志的类型：func(string)
	type saveLog func(msg string)

	//将字符串转换为int64,如果转换失败调用saveLog
	func stringToInt(s string, log saveLog) int64 {

		if value, err := strconv.ParseInt(s, 0, 0); err != nil {
			log(err.Error())
			return 0
		} else {
			return value
		}
	}

	//记录日志消息的具体实现
	func myLog(msg string) {
		fmt.Println("Find Error:", msg)
	}

	func main() {
		stringToInt("123", myLog) //转换时将调用mylog记录日志
		stringToInt("s", myLog)
	}
```
这里我们定义了一个类型，专门用作记录日志的标准接口。在stringToInt函数中如果转换失败则调用我自己定义的接口函数进行日志处理，至于最终执行的哪个函数，则无需关心。

#### defer 延迟函数
defer 又是一个创新，它的作用是：延迟执行，在声明时不会立即执行，而是在函数return后时按照后进先出的原则依次执行每一个defer。这样带来的好处是，能确保我们定义的函数能百分之百能够被执行到，这样就能做很多我们想做的事，如释放资源，清理数据，记录日志等

这里我们重点来说明下defer的执行顺序
```golang
	func deferFunc() int {
		index := 0

		fc := func() {

			fmt.Println(index, "匿名函数1")
			index++

			defer func() {
				fmt.Println(index, "匿名函数1-1")
				index++
			}()
		}

		defer func() {
			fmt.Println(index, "匿名函数2")
			index++
		}()

		defer fc()

		return func() int {
			fmt.Println(index, "匿名函数3")
			index++
			return index
		}()
	}

	func main() {
		deferFunc()
	}
```
这里输出结果如下，

	0 匿名函数3
	1 匿名函数1
	2 匿名函数1-1
	3 匿名函数2

有如下结论：

+ defer 是在执行完return 后执行
+ defer 后进先执行

另外，我们常使用defer去关闭IO,在正常打开文件后，就立刻声明一个defer，这样就不会忘记关闭文件，也能保证在出现异常等不可预料的情况下也能关闭文件。而不像其他语言：`try-catch` 或者 `using()` 方式进行处理。
```golang
	file , err :=os.Open(file)
	if err != nil {
		return err
	}
	defer file.Close()
	//dosomething with file
```

后续，我将讨论： 作用域、传值和传指针 以及 保留函数init(),main()

本笔记中所写代码存储位置：

+ [defer.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/defer.go)
+ [defineFunctionType.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/defineFunctionType.go)
+ [function.go](https://github.com/devYu/GoLangStudy/tree/master/codeDemo/function.go)


[上篇-控制语句](https://github.com/devYu/GoLangStudy/blob/master/myNotes/controlStructures.md)
