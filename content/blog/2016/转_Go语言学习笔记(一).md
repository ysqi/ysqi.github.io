
---
date: 2016-12-31T11:33:21+08:00
title: "Go语言学习笔记(一)"
description: ""
disqus_identifier: 1485833601608649675
slug: "Goyu-yan-xue-xi-bi-ji-(yi-)"
source: "https://segmentfault.com/a/1190000006047562"
tags: 
- go语言 
- golang 
topics:
- 编程语言与开发
---

> 主要是看[《the way to
> go》](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/preface.md)时的一些笔记，比较凌乱，内容也不全，以后慢慢补充。

标签（空格分隔）： go 监控

------------------------------------------------------------------------

### 关键字

`break` `default` `func` `interface` `select` `case` `defer` `go` `map`
`struct` `chan` `else` `goto`\
`package` `switch` `const` `fallthrough` `if` `range` `type` `continue`
`for` `import` `return` `var`

### 基本类型，内置函数

`append` `bool` `byte` `cap` `close` `complex` `complex64` `complex128`
`uint16` `copy` `false` `float32` `float64`\
`imag` `int` `int8` `int16` `uint32` `int32` `int64` `iota` `len` `make`
`new` `nil` `panic` `uint64` `print` `println`\
`real` `recover` `string` `true` `uint` `uint8` `uintptr`

### 可见性

首字母大写相当于public，小写相当于pivate

### 包重命名

    package main

    import fm "fmt" // alias3

    func main() {
       fm.Println("hello, world")
    }

### global virable

declare after package import

### func params and return values

    func Sum(a, b int) int { return a + b }

    func functionName(parameter_list) (return_value_list) {
       …
    }

    //????????????
    func (t T) Method1() {
       //...
    }

parameter\_list 的形式为 (param1 type1, param2 type2, …)\
return\_value\_list 的形式为 (ret1 type1, ret2 type2, …)

### 常量定义

可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。常量的定义格式`const identifier [type] = value`

-   显式类型定义： `const b string = "abc"`

-   隐式类型定义： `const b = "abc"`

> 常量的值必须是能够在编译时就能够确定的，数字型的常量是没有大小和符号的，并且可以使用任何精度而不会导致溢出

### 变量

    var a, b *int
    var a int
    var b bool
    var str string
    var (
        a int
        b bool
        str string
    )

> 变量被声明之后，系统自动赋予它该类型的零值：int 为 0，float 为
> 0.0，bool 为 false，string 为空字符串，指针为 nil。

编译时即可推断类型

    var identifier [type] = value
    var a int = 15
    var i = 5
    var b bool = false
    var str string = "Go says hello to the world!"

    //编译时推断类型
    var a = 15
    var b = false
    var str = "Go says hello to the world!"

    var (
        a = 15
        b = false
        str = "Go says hello to the world!"
        numShips = 50
        city string
    )

    //运行时自动推断
    var (
        HOME = os.Getenv("HOME")
        USER = os.Getenv("USER")
        GOROOT = os.Getenv("GOROOT")
    )

> 函数内部变量首字母大写全局or局部:**函数内部变量均为局部**

### 基本类型何运算符

### 数字类型

**整数**\
int8（-128 -&gt; 127）\
int16（-32768 -&gt; 32767）\
int32（-2,147,483,648 -&gt; 2,147,483,647）\
int64（-9,223,372,036,854,775,808 -&gt; 9,223,372,036,854,775,807）

**无符号整数**\
uint8（0 -&gt; 255）\
uint16（0 -&gt; 65,535）\
uint32（0 -&gt; 4,294,967,295）\
uint64（0 -&gt; 18,446,744,073,709,551,615）

**浮点型（IEEE-754 标准）**\
float32（+- 1e-45 -&gt; +- 3.4 \* 1e38）\
float64（+- 5 *1e-324 -&gt; 107* 1e308）

> float32 精确到小数点后 7 位，float64 精确到小数点后 15
> 位。通过增加前缀 0 来表示 8 进制数（如：077），增加前缀 0x 来表示 16
> 进制数（如：0xFF），以及使用 e 来表示 10 的连乘（如： 1e3 = 1000，或者
> 6.022e23 = 6.022 x 1e23）。

    package main

    import "fmt"

    func main() {
        var n int16 = 34
        var m int32
        // compiler error: cannot use n (type int16) as type int32 in assignment
        //m = n
        m = int32(n)

        fmt.Printf("32 bit int is: %d\n", m)
        fmt.Printf("16 bit int is: %d\n", n)
    }

**格式化输出**\
`%d` 用于格式化整数（`%x` 和 `%X` 用于格式化 16 进制表示的数字），`%g`
用于格式化浮点型（`%f` 输出浮点数，`%e` 输出科学计数表示法），`%0d`
用于规定输出定长的整数，其中开头的数字 0 是必须的。`%n.mg` 用于表示数字
n 并精确到小数点后 m 位，除了使用 g 之外，还可以使用 e 或者
f，例如：使用格式化字符串 `%5.2e` 来输出 3.4 的结果为 `3.40e+00`。

**类型转换**

    func IntFromFloat64(x float64) int {
        if math.MinInt32 <= x && x <= math.MaxInt32 { // x lies in the integer range
            whole, fraction := math.Modf(x)
            if fraction >= 0.5 {
                whole++
            }
            return int(whole)
        }
        panic(fmt.Sprintf("%g is out of the int32 range", x))
    }

**运算符优先级**

    优先级   运算符
     7      ^ !
     6      * / % << >> & &^
     5      + - | ^
     4      == != < <= >= >
     3      <-
     2      &&
     1      ||

### 字符串

**解释字符串**\
该类字符串使用双引号括起来，其中的相关的转义字符将被替换，这些转义字符包括：

    \n：换行符
    \r：回车符
    \t：tab 键
    \u 或 \U：Unicode 字符
    \\：反斜杠自身

**非解释字符串**\
该类字符串使用反引号括起来，支持换行，例如：\
`This is a raw string \n` 中的 `\n` 会被原样输出。

> 在循环中使用加号+拼接字符串并不是最高效的做法，更好的办法是使用函数
> strings.Join()，有没有更好地办法了？有！使用字节缓冲（bytes.Buffer）拼接更加给力！

**字符串操作**\
`strings和strconv包`

    //前缀和后缀
    strings.HasPrefix(s, prefix string) bool
    strings.HasSuffix(s, suffix string) bool
    //包含关系
    strings.Contains(s, substr string) bool
    strings.ContainsAny(s, chars string) bool
    strings.ContainsRune(s string, r rune) bool
    //索引
    strings.Index(s, str string) int
    strings.LastIndex(s, str string) int
    strings.IndexRune(s string, r rune) int
    //替换
    strings.Replace(str, old, new, n) string
    //次数统计
    strings.Count(s, str string) int
    //重复次数
    strings.Repeat(s, count int) string
    //大小写
    strings.ToLower(s) string
    strings.ToLower(s) string
    //修剪字符串
    func Trim(s string, cutset string) string
    func TrimLeft(s string, cutset string) string
    func TrimRight(s string, cutset string) string
    func TrimSpace(s string) string
    //分割字符串
    strings.Fields(s) []string
    strings.Split(s, sep string) []string
    //拼接字符串
    strings.Join(sl []string, sep string) string

**日期和时间**

    func timeTest(){
        t := time.Now()
        fm.Println(t)
        fm.Println(t.Day(), t.Hour(), t.Minute())
        t = time.Now().UTC()
        fm.Println(t)
        fm.Println(t.Format(time.RFC822))
        fm.Println(t.Format(time.ANSIC))
        fm.Println(t.Format("02 Jan 2006 15:04"))
    }

**指针**

    //声明方式
    var intP *int
    //使用方式
    intP = &i1

不允许指针运算`pointer+2`、`c=*p++`，空指针的反向引用是不合法。

**控制结构**

-   if-esle语句

-   switch语句\
    不需要break，执行完分支自动退出switch，如果需要继续执行则使用fallthrough

<!-- -->

    switch num1 {
    case 98, 99:
        fmt.Println("It's equal to 98")
    case 100: 
        fmt.Println("It's equal to 100")
    default:
        fmt.Println("It's not equal to 98 or 100")
    }

    //多条件同行，fallthrough，return跳出
    func switchStruct() int {
        a := 4
        switch a {
        case 1,2,3: fmt.Println(a)
        case 4: fallthrough
        case 5:
            fm.Println("break")
            return 1
        default:
            fmt.Println("ALL")
        }
        return 0
    }

    //bool用法
    switch {
        case num1 < 0:
            fmt.Println("Number is negative")
        case num1 > 0 && num1 < 10:
            fmt.Println("Number is between 0 and 10")
        default:
            fmt.Println("Number is 10 or greater")
    }

    //初始化用法
    switch a, b := x[i], y[j]; {
        case a < b: t = -1
        case a == b: t = 0
        case a > b: t = 1
    }

-   for语句

<!-- -->

    func forStatement()  {
        for i := 1; i < 15; i++{
            fm.Println(i)
        }

        j := 1
        GO:
        fm.Println(j)
        j++
        if j < 15{
            goto GO
        }

        for k := 0; k < 5; k++ {
            for m := 0; m < k; m++ {
                fm.Printf("G")
            }
            fm.Println()
        }

        str := "GGGGGGGGGGGGGGGGGGGGGGGG"
        for i := 0; i < 5; i++ {
            fm.Println(str[0:i+1])
        }

        str1 := "G"
        for i := 0; i < 5; i++ {
            fm.Println(str1)
            str1 += "G"
        }
    }

### 函数

Go 里面有三种类型的函数：

1.  普通的带有名字的函数

2.  匿名函数或者lambda函数

3.  方法（Methods）

**defer关键字**

> 在函数return之后执行相关语句，多个defer逆序执行

    func a() {
        i := 0
        defer fmt.Println(i)
        i++
        return
    }
    //输出0，输出语句之前的值

-   关闭文件流：\
    // open a file defer file.Close() （详见第 12.2 节）

-   解锁一个加锁的资源\
    mu.Lock() defer mu.Unlock() （详见第 9.3 节）

-   打印最终报告\
    printHeader() defer printFooter()

-   关闭数据库链接\
    // open a database connection defer disconnectFromDB()

<!-- -->

    func testDefer()  {
        fmt.Printf("In function1 at the top\n")
        defer func2()
        fmt.Printf("In function1 at the bottom!\n")
        i := 0
        i++
        defer fmt.Println(i)
        i++
        for i := 0; i < 5; i++ {
            defer fmt.Printf("%d", i)
        }

        defer deferC(deferA(), deferB())
        fm.Println("Begin:\n")

        return
    }

    func deferA() (rtn string) {
        fm.Println("in A")
        rtn = "AA"
        tmp := "tmp1"
        defer func() {
            fm.Println(tmp)
        }()
        defer fm.Println(rtn)
        tmp = "tmp2"
        return "AAAA"
    }

    func deferB() string {
        fm.Println("in B")
        return "B"
    }

    func deferC(str1, str2 string)  {
        fm.Println("in C")
    }
    /*输出
    In function1 at the top
    In function1 at the bottom!
    in A
    AA
    tmp2
    in B
    Begin:

    in C
    432101
    function2: Deferred until the end of the calling function!
    */

> `defer`函数中的函数（`defer deferC(deferA(), deferB())`中的`A`和`B`）会先于return执行，`defer func()`可以`rentun`语句执行完变量的最新的值，无`func()`时则函数参数取对应`defer`语句执行完变量的最新值。

**内置函数**\

**递归函数**

**函数作参数**

    func main() {
        callback(1, Add)
    }

    func Add(a, b int) {
        fmt.Printf("The sum of %d and %d is: %d\n", a, b, a+b)
    }

    func callback(y int, f func(int, int)) {
        f(y, 2) // this becomes Add(1, 2)
    }

**闭包**

    func main() {
        var f = Adder()
        fmt.Print(f(1), " - ")
        fmt.Print(f(20), " - ")
        fmt.Print(f(300))
    }

    func Adder() func(int) int {
        var x int
        return func(delta int) int {
            x += delta
            return x
        }
    }
    // 1,21,321

### 数组与切片

**数组**

Go 语言中的数组是一种**值类型**（不像 C/C++
中是指向首元素的指针），所以可以通过 new() 来创建： var arr1 =
new(\[5\]int)。**arr1 的类型是 \*\[5\]int，而 arr2的类型是
\[5\]int**。结果就是当把一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。例如：

    arr2 := arr1
    arr2[2] = 100

两个数组就有了不同的值，在赋值后修改 arr2 不会对 arr1 生效。

    func f(a [3]int) { fmt.Println(a) }
    func fp(a *[3]int) { fmt.Println(a) }

    func main() {
        var ar [3]int
        f(ar)   // passes a copy of ar
        fp(&ar) // passes a pointer to ar
    }

> 当数组赋值时，发生了数组内存拷贝。

**切片**\
切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率。

    var identifier []type
    var slice1 []type = arr1[start:end]
    var slice1 []type = arr1[:] //全部
    slice1 = &arr1              //全部

> 一个由数字 1、2、3
> 组成的切片可以这么生成：`s := [3]int{1,2,3} `甚至更简单的
> `s := []int{1,2,3}`。**数组区别:数组是切片的特例**

**buffer 串联字符串**\
创建一个 buffer，通过 `buffer.WriteString(s)` 方法将字符串 s
追加到后面，最后再通过 `buffer.String()` 方法转换为
string，这种实现方式比使用 `+=` 要更节省内存和 CPU。

    var buffer bytes.Buffer
    for {
        if s, ok := getNextString(); ok { //method getNextString() not shown here
            buffer.WriteString(s)
        } else {
            break
        }
    }
    fmt.Print(buffer.String(), "\n")

    func AppendByte(slice []byte, data ...byte) []byte {
        m := len(slice)
        n := m + len(data)
        if n > cap(slice) { // if necessary, reallocate
            // allocate double what's needed, for future growth.
            newSlice := make([]byte, (n+1)*2)
            copy(newSlice, slice)
            slice = newSlice
        }
        slice = slice[0:n]
        copy(slice[m:n], data)
        return slice
    }

**for-range**

    seasons := []string{"Spring", "Summer", "Autumn", "Winter"}
    //index&val
    for ix, season := range seasons {
        fmt.Printf("Season %d is: %s\n", ix, season)
    }
    //just val
    var season string
    for _, season = range seasons {
        fmt.Printf("%s\n", season)
    }
    //just index
    for ix := range seasons {
        fmt.Printf("%d", ix)
    }

**复制与追加**

    func main() {
        // count number of characters:
        str1 := "asSASA ddd dsjkdsjs dk"
        fmt.Printf("The number of bytes in string str1 is %d\n",len(str1))
        fmt.Printf("The number of characters in string str1 is %d\n",utf8.RuneCountInString(str1))
        str2 := "asSASA ddd dsjkdsjsこん dk"
        fmt.Printf("The number of bytes in string str2 is %d\n",len(str2))
        fmt.Printf("The number of characters in string str2 is %d",utf8.RuneCountInString(str2))
    }
    /* Output:
    The number of bytes in string str1 is 22
    The number of characters in string str1 is 22
    The number of bytes in string str2 is 28
    The number of characters in string str2 is 24
    */

    //将一个字符串追加到某一个字符数组的尾部
    var b []byte
    var s string
    b = append(b, s...)

> Go
> 语言中的字符串是不可变的,必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式。

    s := "hello"
    c := []byte(s)
    c[0] = ’c’
    s2 := string(c) // s2 == "cello"

**append操作**

-   将切片 b 的元素追加到切片 a 之后：`a = append(a, b...)`

-   复制切片 a 的元素到新的切片 b 上：

<!-- -->

    b = make([]T, len(a))
    copy(b, a)

-   删除位于索引 i 的元素：\
    `a = append(a[:i], a[i+1:]...)`

-   切除切片 a 中从索引 i 至 j 位置的元素：\
    `a = append(a[:i], a[j:]...)`

-   为切片 a 扩展 j 个元素长度：\
    `a = append(a, make([]T, j)...)`

-   在索引 i 的位置插入元素 x：\
    `a = append(a[:i], append([]T{x}, a[i:]...)...)`

-   在索引 i 的位置插入长度为 j 的新切片：\
    `a = append(a[:i], append(make([]T, j), a[i:]...)...)`

-   在索引 i 的位置插入切片 b 的所有元素：\
    `a = append(a[:i], append(b, a[i:]...)...)`

-   取出位于切片 a 最末尾的元素 x：\
    `x, a = a[len(a)-1], a[:len(a)-1]`

-   将元素 x 追加到切片 a：\
    `a = append(a, x)`

**垃圾回收**

    var digitRegexp = regexp.MustCompile("[0-9]+")

    func FindDigits(filename string) []byte {
        b, _ := ioutil.ReadFile(filename)
        return digitRegexp.Find(b)
    }
    // 通过拷贝避免小切片占用大内存（整个文件）
    func FindDigits(filename string) []byte {
       b, _ := ioutil.ReadFile(filename)
       b = digitRegexp.Find(b)
       c := make([]byte, len(b))
       copy(c, b)
       return c
    }

> `error: rune is not a Type`:代码中又于rune重名的函数

### Map

`var map1 map[keytype]valuetype`

> 不要使用 new，永远用 make 来构造 map

**map类型是非线程安全的，并行访问map数据会出错。并且为了性能map没有锁机制**

    // map排序
    var (
    barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,"delta": 87, "echo": 56, "foxtrot": 12,"golf": 34, "hotel": 16, "indio": 87,"juliet": 65, "kili": 43, "lima": 98}
    )

    func main() {
        fmt.Println("unsorted:")
        for k, v := range barVal {
            fmt.Printf("Key: %v, Value: %v / ", k, v)
        }
        keys := make([]string, len(barVal))
        i := 0
        for k, _ := range barVal {
            keys[i] = k
        i++
        }
        sort.Strings(keys)
        fmt.Println()
        fmt.Println("sorted:")
        for _, k := range keys {
            fmt.Printf("Key: %v, Value: %v / ", k, barVal[k])
        }
    }

### 包（package）

**regexp包**\
**锁和sync包**\
**big包**

### struct和method

可以使用 make() 的三种类型：\
`slices  /  maps / channels（见第 14 章）`

> 在一个结构体中对于每一种数据类型只能有一个匿名字段。

**内嵌结构体**

    package main

    import "fmt"

    type A struct {
        ax, ay int
    }

    type B struct {
        A
        bx, by float32
    }

    func main() {
        b := B{A{1, 2}, 3.0, 4.0}
        fmt.Println(b.ax, b.ay, b.bx, b.by)
        fmt.Println(b.A)
    }

定义方法的一般格式：\
`func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }`

结构体方法

    type TwoInts struct {
        a int
        b int
    }

    func main() {
        two1 := new(TwoInts)
        two1.a = 12
        two1.b = 10

        fmt.Printf("The sum is: %d\n", two1.AddThem())
        fmt.Printf("Add them to the param: %d\n", two1.AddToParam(20))

        two2 := TwoInts{3, 4}
        fmt.Printf("The sum is: %d\n", two2.AddThem())
    }

    func (tn *TwoInts) AddThem() int {
        return tn.a + tn.b
    }

    func (tn *TwoInts) AddToParam(param int) int {
        return tn.a + tn.b + param
    }

非结构体类型方法

    type IntVector []int

    func (v IntVector) Sum() (s int) {
        for _, x := range v {
            s += x
        }
        return
    }

    func main() {
        fmt.Println(IntVector{1, 2, 3}.Sum()) // 输出是6
    }

> 类型和作用在它上面定义的方法可以不同文件，但是必须在同一个包里定义，除非定义别名：

    type myTime struct {
        time.Time //anonymous field
    }
    func (t myTime) first3Chars() string {
        return t.Time.String()[0:3]
    }
    func main() {
        m := myTime{time.Now()}
        // 调用匿名Time上的String方法
        fmt.Println("Full time now:", m.String())
        // 调用myTime.first3Chars
        fmt.Println("First 3 chars:", m.first3Chars())
    }
    /* Output:
    Full time now: Mon Oct 24 15:34:54 Romance Daylight Time 2011
    First 3 chars: Mon
    */

指针方法和值方法都可以在指针或非指针上被调用

    type List []int

    func (l List) Len() int        { return len(l) }
    func (l *List) Append(val int) { *l = append(*l, val) }

    func main() {
        // 值
        var lst List
        lst.Append(1)
        fmt.Printf("%v (len: %d)", lst, lst.Len()) // [1] (len: 1)

        // 指针
        plst := new(List)
        plst.Append(2)
        fmt.Printf("%v (len: %d)", plst, plst.Len()) // &[2] (len: 1)
    }

**并发访问对象**

    import  “sync”

    type Info struct {
        mu sync.Mutex
        // ... other fields, e.g.: Str string
    }

    func Update(info *Info) {
        info.mu.Lock()
        // critical section:
        info.Str = // new value
        // end critical section
        info.mu.Unlock()
    }

**内嵌方法**

    //内嵌方法测试
    func methodTest() {
        point := &NamedPoint{Point{3, 4}, "name"}
        fm.Println(point.Abs(), point.Point.Abs())
    }

    type Point struct {
        x, y float64
    }

    func (p *Point)Abs() float64 {
        return math.Sqrt(p.x*p.x + p.y*p.y)
    }

    type NamedPoint struct {
        Point
        name string
    }

    func (p *NamedPoint)Abs() float64 {
        return p.Point.Abs() * p.Point.Abs()
    }

**多重继承**

    type Camera struct{}

    func (c *Camera) TakeAPicture() string {
        return "Click"
    }

    type Phone struct{}

    func (p *Phone) Call() string {
        return "Ring Ring"
    }

    type CameraPhone struct {
        Camera
        Phone
    }

    func main() {
        cp := new(CameraPhone)
        fmt.Println("Our new CameraPhone exhibits multiple behaviors...")
        fmt.Println("It exhibits behavior of a Camera: ", cp.TakeAPicture())
        fmt.Println("It works like a Phone too: ", cp.Call())
    }

**垃圾回收和SetFinalizer**\
内存状态

    // fmt.Printf("%d\n", runtime.MemStats.Alloc/1024)
    // 此处代码在 Go 1.5.1下不再有效，更正为
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("%d Kb\n", m.Alloc / 1024)

回收对象时，对象移除前操作：

    runtime.SetFinalizer(obj, func(obj *typeObj))

### 接口和反射

不同的结构实现interface中定义的接口，通过接口变量自动识别对应结构来实现多态，所有类型必须完全实现interface中的所有函数

    type Shaper interface {
        Area() float32
    }

    type Square struct {
        side float32
    }

    func (sq *Square) Area() float32 {
        return sq.side * sq.side
    }

    type Rectangle struct {
        length, width float32
    }

    func (r Rectangle) Area() float32 {
        return r.length * r.width
    }

    func main() {

        r := Rectangle{5, 3} // Area() of Rectangle needs a value
        q := &Square{5}      // Area() of Square needs a pointer
        // shapes := []Shaper{Shaper(r), Shaper(q)}
        // or shorter
        shapes := []Shaper{r, q}
        fmt.Println("Looping through shapes for area ...")
        for n, _ := range shapes {
            fmt.Println("Shape details: ", shapes[n])
            fmt.Println("Area of this shape is: ", shapes[n].Area())
        }
    }

> 接口的实现通过指针则变量必须传入指针；接口可以嵌套。

**类型判断**

    switch t := areaIntf.(type) {
    case *Square:
        fmt.Printf("Type Square %T with value %v\n", t, t)
    case *Circle:
        fmt.Printf("Type Circle %T with value %v\n", t, t)
    case nil:
        fmt.Printf("nil value: nothing to check?\n")
    default:
        fmt.Printf("Unexpected type %T\n", t)
    }

    //简洁版
    switch areaIntf.(type) {
    case *Square:
        // TODO
    case *Circle:
        // TODO
    ...
    default:
        // TODO
    }

> 也可用于测试变量是否实现某接口

    type Stringer interface {
        String() string
    }

    if sv, ok := v.(Stringer); ok {
        fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
    }

**接口调用**

-   指针方法可以通过指针调用

-   值方法可以通过值调用

-   接收者是值的方法可以通过指针调用，因为指针会首先被解引用

-   接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址

-   类型 *T 的可调用方法集包含接受者为* T 或 T 的所有方法集

-   类型 T 的可调用方法集包含接受者为 T 的所有方法

-   类型 T 的可调用方法集不包含接受者为 \*T 的方法

**自定义类型排序接口**

    package sort

    // 排序接口
    type Sorter interface {
        Len() int
        Less(i, j int) bool
        Swap(i, j int)
    }

    func Sort(data Sorter)  {
        len := data.Len()
        for i := 1; i < len; i++{
            for j := 0; j < len - i; j++ {
                if data.Less(j+1, j) {
                    data.Swap(j+1, j)
                }
            }
        }
    }

    func main(){
        Sunday    := day{0, "SUN", "Sunday"}
        Monday    := day{1, "MON", "Monday"}
        Tuesday   := day{2, "TUE", "Tuesday"}
        Wednesday := day{3, "WED", "Wednesday"}
        Thursday  := day{4, "THU", "Thursday"}
        Friday    := day{5, "FRI", "Friday"}
        Saturday  := day{6, "SAT", "Saturday"}
        
        data2 := []day{Tuesday, Thursday, Wednesday, Sunday, Monday, Friday, Saturday, Thursday}
        array2 := dayArr(data2)
        sort.Sort(array2)
        for _, d := range data2 {
            fmt.Printf("%v ", d)
        }
        fmt.Printf("\n")
        fm.Println(array2)
        fmt.Printf("\n")
    }

    // 自己想到的实现方式
    type day struct {
        num int
        shortName string
        longName string
    }

    type dayArr []day

    func (p dayArr)Len() int {
        return len(p)
    }

    func (p dayArr)Less(i, j int) bool {
        return p[i].num < p[j].num
    }

    func (p dayArr)Swap(i, j int) {
        p[i], p[j] = p[j], p[i]
    }

    func main(){
        Sunday    := day{0, "SUN", "Sunday"}
        Monday    := day{1, "MON", "Monday"}
        Tuesday   := day{2, "TUE", "Tuesday"}
        Wednesday := day{3, "WED", "Wednesday"}
        Thursday  := day{4, "THU", "Thursday"}
        Friday    := day{5, "FRI", "Friday"}
        Saturday  := day{6, "SAT", "Saturday"}
        
        data3 := []*day{&Tuesday, &Thursday, &Wednesday, &Sunday, &Monday, &Friday, &Saturday}
        // 结构体初始化
        array3 := dayArray{data3}
        sort.Sort(&array3)
        for _, d := range data3 {
            fmt.Printf("%v ", d)
        }
        fmt.Printf("\n")
        fm.Println(array3)
        fmt.Printf("\n")
    }

    // 作者实现方式
    type dayArray struct {
        data []*day
    }

    func (p *dayArray)Len() int {
        return len(p.data)
    }

    func (p *dayArray)Less(i, j int) bool {
        return p.data[i].num < p.data[j].num
    }

    func (p *dayArray)Swap(i, j int) {
        p.data[i], p.data[j]= p.data[j], p.data[i]
    }

> 两种方式对比:

-   第一种方式使用struct数组(切片更恰当)和变量，都是通过值拷贝；第二种通过struct指针数组和数组的指针，通过引用。故内存占用和调用消耗有差别。

-   暂时没想到，以后补充

**构建通用类型或包含不同类型变量的数组**\
对于返回值未知的接口，可以通过返回空接口。

    package min

    type Miner interface {
        Len() int
        ElemIx(ix int) interface{}
        Less(i, j int) bool
    }

    func Min(data Miner) interface{}  {
        min := data.ElemIx(0)
        for i:=1; i < data.Len(); i++ {
            if data.Less(i, i-1) {
                    min = data.ElemIx(i)
            }
        }
        return min
    }

    type IntArray []int
    func (p IntArray) Len() int { return len(p) }
    func (p IntArray) ElemIx(ix int) interface{} { return p[ix] }
    func (p IntArray) Less(i, j int) bool { return p[i] < p[j] }

    type StringArray []string
    func (p StringArray) Len() int { return len(p) }
    func (p StringArray) ElemIx(ix int) interface{} { return p[ix] }
    func (p StringArray) Less(i, j int) bool { return p[i] < p[j] }

**复制数据切片至空接口切片**\
内存布局不一样，需要一个个赋值

    var dataSlice []myType = FuncReturnSlice()
    var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
    for ix, d := range dataSlice {
        interfaceSlice[ix] = d
    }

**通用节点数据结构**

    type Node struct {
        le   *Node
        data interface{}
        ri   *Node
    }

    func NewNode(left, right *Node) *Node {
        return &Node{left, nil, right}
    }

    func (n *Node) SetData(data interface{}) {
        n.data = data
    }

**接口和动态类型**

**函数重载**\
通过函数参数`...T`实现，或空接口\
`fmt.Printf(format string, a ...interface{}) (n int, errno error)`

**接口继承**\
当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法。

    type Task struct {
        Command string
        *log.Logger
    }

    func NewTask(command string, logger *log.Logger) *Task {
        return &Task{command, logger}
    }

    task.Log()

    //多重继承
    type ReaderWriter struct {
        *io.Reader
        *io.Writer
    }

**类型转换问题**\
[对空接口interface{}进行强制类型转换报错](http://stackoverflow.com/questions/18041334/convert-interface-to-int-in-go-lang)

> Conversions are expressions of the form T(x) where T is a type and x
> is an expression that can be converted to type T.

A non-constant value x can be converted to type T in any of these cases:

1.  x is assignable to T.

2.  x's type and T have identical underlying types.

3.  x's type and T are unnamed pointer types and their pointer base
    types have identical underlying types.

4.  x's type and T are both integer or floating point types.

5.  x's type and T are both complex types.

6.  x is an integer or a slice of bytes or runes and T is a string type.

7.  x is a string and T is a slice of bytes or runes.

> 转换是形如`T(x)`的表达式，其中`T`是一种类型，而`x`是能转换为类型`T`的表达式

下面任何一种情况中非常量x都能转换为类型`T`

1.  x可赋值给T

2.  x的类型和T的底层类型一致

3.  x的类型和T是匿名指针类型且它们的指针基类型的底层类型一致

4.  x的类型和T都是整型或浮点型

5.  x的类型和T都是复数类型

6.  x是整数、bytes切片或runes，且T是string

7.  x是string且T是bytes切片或runes

### GO中的OO

OO 语言最重要的三个方面分别是：封装，继承和多态，在 Go
中它们是怎样表现的呢？

1.  封装（数据隐藏）：Go 把它从4层简化为了2层:

-   包范围内的：通过标识符首字母小写，对象只在它所在的包内可见

-   可导出的：通过标识符首字母大写，对象 对所在包以外也可见

1.  继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现

2.  多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。

### 读写数据

**输入输出**\
**文件读写**

    inputFile, inputError := os.Open("input.dat")
    if inputError != nil {
        fmt.Printf("An error occurred on opening the inputfile\n" +
            "Does the file exist?\n" +
            "Have you got acces to it?\n")
        return // exit the function on error
    }
    defer inputFile.Close()

    inputReader := bufio.NewReader(inputFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        if readerError == io.EOF {
            return
        }
        fmt.Printf("The input was: %s", inputString)
    }

**读到字符串**

    func main() {
        inputFile := "products.txt"
        outputFile := "products_copy.txt"
        buf, err := ioutil.ReadFile(inputFile)
        if err != nil {
            fmt.Fprintf(os.Stderr, "File Error: %s\n", err)
            // panic(err.Error())
            }
        fmt.Printf("%s\n", string(buf))
        err = ioutil.WriteFile(outputFile, buf, 0644) // oct, not hex
        if err != nil {
            panic(err. Error())
        }
    }

**缓冲读取**

    buf := make([]byte, 1024)
    ...
    n, err := inputReader.Read(buf)
    if (n == 0) { break}

**按列读取**

    func main() {
        file, err := os.Open("products2.txt")
        if err != nil {
            panic(err)
        }
        defer file.Close()

        var col1, col2, col3 []string
        for {
            var v1, v2, v3 string
            _, err := fmt.Fscanln(file, &v1, &v2, &v3)
            // scans until newline
            if err != nil {
                break
            }
            col1 = append(col1, v1)
            col2 = append(col2, v2)
            col3 = append(col3, v3)
        }
        fmt.Println(col1)
        fmt.Println(col2)
        fmt.Println(col3)
    }

**写文件**

    func main () {
        outputFile, outputError := os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)
        if outputError != nil {
            fmt.Printf("An error occurred with file opening or creation\n")
            return  
        }
        defer outputFile.Close()

        outputWriter := bufio.NewWriter(outputFile)
        outputString := "hello world!\n"

        for i:=0; i<10; i++ {
            outputWriter.WriteString(outputString)
        }
        outputWriter.Flush()
    }

-   os.O\_RDONLY：只读

-   os.O\_WRONLY：只写

-   os.O\_CREATE：创建：如果指定文件不存在，就创建该文件。

-   os.O\_TRUNC：截断：如果指定文件已存在，就将该文件的长度截为0。

<!-- -->

    // 非缓冲写入
    func main() {
        os.Stdout.WriteString("hello, world\n")
        f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0)
        defer f.Close()
        f.WriteString("hello, world in a file\n")
    }

**文件拷贝**

    func CopyFile(dstName, srcName string) (written int64, err error) {
        src, err := os.Open(srcName)
        if err != nil {
            return
        }
        defer src.Close()

        dst, err := os.OpenFile(dstName, os.O_WRONLY|os.O_CREATE, 0644)
        if err != nil {
            return
        }
        defer dst.Close()

        return io.Copy(dst, src)
    }

**读取参数**

    func main() {
        who := "Alice "
        if len(os.Args) > 1 {
            who += strings.Join(os.Args[1:], " ")
        }
        fmt.Println("Good Morning", who)
    }

**JSON**

1.  JSON 与 Go 类型对应如下

-   bool 对应 JSON 的 booleans

-   float64 对应 JSON 的 numbers

-   string 对应 JSON 的 strings

-   nil 对应 JSON 的 null

1.  不是所有的数据都可以编码为JSON类型：只有验证通过的数据结构才能被编码：

-   JSON 对象只支持字符串类型的 key；要编码一个 Go map 类型，map 必须是
    map\[string\]T（T是json包中支持的任何类型）

-   Channel，复杂类型和函数类型不能被编码

-   不支持循环数据结构；它将引起序列化进入一个无限循环指针可以被编码，实际上是对指针指向的值进行编码（或者指针是
    nil）

### 错误处理与测试

### 协程与通道

> 不要通过共享内存来通信，而通过通信来共享内存。

    func main() {
        runtime.GOMAXPROCS(2)
        ch1 := make(chan int)
        ch2 := make(chan int)
        
        go pump1(ch1)
        go pump2(ch2)
        go suck(ch1, ch2)
        
        time.Sleep(1e9)
    }

    func pump1(ch chan int) {
        for i:=0; ; i++ {
            ch <- i*2
        }
    }

    func pump2(ch chan int) {
        for i:=0; ; i++ {
            ch <- i+5
        }
    }

    func suck(ch1,ch2 chan int) {
        for i := 0; ; i++ {
            select {
                case v := <- ch1:
                    fmt.Printf("%d - Received on channel 1: %d\n", i, v)
                case v := <- ch2:
                    fmt.Printf("%d - Received on channel 2: %d\n", i, v)
            }
        }
    }

**协程间通信**\
通道声明：

    var identifier chan type
    id := make(chan type)

    // 缓冲通道
    iden := make(chan type, len)

1.  流向通道（发送）\
    `ch <- int1`表示：用通道 ch 发送变量int1（双目运算符，中缀 = 发送）

2.  从通道流出（接收），三种方式：

-   `int2 = <- ch` 表示：变量 int2 从通道 ch（一元运算的前缀操作符，前缀
    = 接收）接收数据（获取新值）

-   假设 int2 已经声明过了，如果没有的话可以写成：`int2 := <- ch`

-   &lt;- ch
    可以单独调用获取通道的（下一个）值，当前值会被丢弃，但是可以用来验证，以下代码是合法的：

<!-- -->

    if <- ch != 1000{
        ...
    }

数据传输

    func TestChan() {
        ch := make(chan string)
        go sendData(ch)
        go recvData(ch)
        time.Sleep(1e9)
    }

    func sendData(ch chan  string)  {
        ch <- "hu"
        ch <- "yuan"
        ch <- "sky"
    }

    func recvData(ch chan string)  {
        recv := ""
        for {
            recv = <-ch
            fmt.Println(recv)
        }
    }

**阻塞**

    // 发送阻塞
    func main() {
        ch1 := make(chan int)
        go pump(ch1)       // pump hangs
        time.Sleep(2*1e9)
        fmt.Println("Recv", <-ch1) // prints only 0
        time.Sleep(1e9)
    }

    func pump(ch chan int) {
        for i := 1; ; i++ {
            ch <- i
            fmt.Println("Send ",i)
        }
    }

    //发送阻塞
    func main() {
        c := make(chan int)
        go func() {
            time.Sleep(15 * 1e9)
            x := <-c
            fmt.Println("received", x)
        }()
        fmt.Println("sending", 10)
        c <- 10
        fmt.Println("sent", 10)
    }

**信号量模式**\
循环并行执行

    // 测试同步执行
    type Empty interface {}

    const N =  100000

    func Compare() {
        ch_buf := make(chan Empty, N)
        data := make([]float64, N)
        for i:=0; i < N; i++{
            data[i] = float64(i)
        }
        s := time.Now()
        TestChannal(data, ch_buf)
        fmt.Println(time.Now().Sub(s))
        //fmt.Println(data)

        for i:=0; i < N; i++{
            data[i] = float64(i)
        }
        s = time.Now()
        TestNormal(data)
        fmt.Println(time.Now().Sub(s))
        //fmt.Println(data)
    }

    func TestChannal(data []float64, ch_buf chan Empty) {
        var em Empty
        for ix, val :=range data {
            go func(ix int, val float64) {
                for j := 0; j < N; j++ {
                    data[ix] += val
                }
                ch_buf <- em
            }(ix, val)
        }
        for i := 0; i < N; i++ { <-ch_buf}
    }

    func TestNormal(data []float64){
        for ix, val := range data {
            for j := 0; j < N; j++ {
                data[ix] += val
            }
        }
    }

**通道的方向**

    var send_only chan<- int    // channel can only send data
    var recv_only <-chan int    // channel can onley recv data

> 只接收的通道（`<-chan T`）无法关闭，因为关闭通道是发送者用来表示不再给通道发送值了，所以对只接收通道是没有意义的。

### 网络、模板和网络应用

### 注意事项

-   所有的包名使用小写

-   如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。

-   must define main function in main package else you will get
    error 'undefined'. main function has not params and return value

-   init function execute before main

-   production server must use function in package "fmt"

-   全局变量是允许声明但不使用



