
---
date: 2016-12-31T11:32:51+08:00
title: "编写可测试的Go代码"
description: ""
disqus_identifier: 1485833571499659850
slug: "bian-xie-ke-ce-shi-de-Godai-ma"
source: "https://segmentfault.com/a/1190000007533362"
tags: 
- 单元测试 
- 测试 
- golang 
topics:
- 编程语言与开发
---

原文链接：[http://tabalt.net/blog/golang...](http://tabalt.net/blog/golang-testing/)

Golang作为一门标榜工程化的语言，提供了非常简便、实用的编写单元测试的能力。本文通过Golang源码包中的用法，来学习在实际项目中如何编写可测试的Go代码。

### 第一个测试 “Hello Test!”

首先，在我们`$GOPATH/src`目录下创建hello目录，作为本文涉及到的所有示例代码的根目录。

然后，新建名为hello.go的文件，定义一个函数hello()，功能是返回一个由若干单词拼接成句子：

    package hello

    func hello() string {
        words := []string{"hello", "func", "in", "package", "hello"}
        wl := len(words)

        sentence := ""
        for key, word := range words {
            sentence += word
            if key < wl-1 {
                sentence += " "
            } else {
                sentence += "."
            }
        }
        return sentence
    }

接着，新建名为hello\_test.go的文件，填入如下内容：

    package hello

    import (
        "fmt"
        "testing"
    )

    func TestHello(t *testing.T) {
        got := hello()
        expect := "hello func in package hello."

        if got != expect {
            t.Errorf("got [%s] expected [%s]", got, expect)
        }
    }

    func BenchmarkHello(b *testing.B) {
        for i := 0; i < b.N; i++ {
            hello()
        }
    }

    func ExampleHello() {
        hl := hello()
        fmt.Println(hl)
        // Output: hello func in package hello.
    }

最后，打开终端，进入hello目录，输入`go test`命令并回车，可以看到如下输出：

    PASS
    ok      hello  0.007s

### 编写测试代码

Golang的测试代码位于某个包的源代码中名称以`_test.go`结尾的源文件里，测试代码包含测试函数、测试辅助代码和示例函数；测试函数有以Test开头的功能测试函数和以Benchmark开头的性能测试函数两种，测试辅助代码是为测试函数服务的公共函数、初始化函数、测试数据等，示例函数则是以Example开头的说明被测试函数用法的函数。

大部分情况下，测试代码是作为某个包的一部分，意味着它可以访问包中不可导出的元素。但在有需要的时候（如避免循环依赖）也可以修改测试文件的包名，如`package hello`的测试文件，包名可以设为`package hello_test`。

#### 功能测试函数

功能测试函数需要接收`*testing.T`类型的单一参数t，testing.T
类型用来管理测试状态和支持格式化的测试日志。测试日志在测试执行过程中积累起来，完成后输出到标准错误输出。

下面是从Go标准库摘抄的 testing.T类型的常用方法的用法：

-   测试函数中的某条测试用例执行结果与预期不符时，调用t.Error()或t.Errorf()方法记录日志并标记测试失败

<!-- -->

    # /usr/local/go/src/bytes/compare_test.go
    func TestCompareIdenticalSlice(t *testing.T) {
        var b = []byte("Hello Gophers!")
        if Compare(b, b) != 0 {
            t.Error("b != b")
        }
        if Compare(b, b[:1]) != 1 {
            t.Error("b > b[:1] failed")
        }
    }

-   使用t.Fatal()和t.Fatalf()方法，在某条测试用例失败后就跳出该测试函数

<!-- -->

    # /usr/local/go/src/bytes/reader_test.go
    func TestReadAfterBigSeek(t *testing.T) {
        r := NewReader([]byte("0123456789"))
        if _, err := r.Seek(1<<31+5, os.SEEK_SET); err != nil {
            t.Fatal(err)
        }
        if n, err := r.Read(make([]byte, 10)); n != 0 || err != io.EOF {
            t.Errorf("Read = %d, %v; want 0, EOF", n, err)
        }
    }

-   使用t.Skip()和t.Skipf()方法，跳过某条测试用例的执行

<!-- -->

    # /usr/local/go/src/archive/zip/zip_test.go
    func TestZip64(t *testing.T) {
        if testing.Short() {
            t.Skip("slow test; skipping")
        }
        const size = 1 << 32 // before the "END\n" part
        buf := testZip64(t, size)
        testZip64DirectoryRecordLength(buf, t)
    }

-   执行测试用例的过程中通过t.Log()和t.Logf()记录日志

<!-- -->

    # /usr/local/go/src/regexp/exec_test.go
    func TestFowler(t *testing.T) {
        files, err := filepath.Glob("testdata/*.dat")
        if err != nil {
            t.Fatal(err)
        }
        for _, file := range files {
            t.Log(file)
            testFowler(t, file)
        }
    }

-   使用t.Parallel()标记需要并发执行的测试函数

<!-- -->

    # /usr/local/go/src/runtime/stack_test.go
    func TestStackGrowth(t *testing.T) {
        t.Parallel()
        var wg sync.WaitGroup

        // in a normal goroutine
        wg.Add(1)
        go func() {
            defer wg.Done()
            growStack()
        }()
        wg.Wait()

        // ...
    }

#### 性能测试函数

性能测试函数需要接收`*testing.B`类型的单一参数b，性能测试函数中需要循环b.N次调用被测函数。testing.B
类型用来管理测试时间和迭代运行次数，也支持和testing.T相同的方式管理测试状态和格式化的测试日志，不一样的是testing.B的日志总是会输出。

下面是从Go标准库摘抄的 testing.B类型的常用方法的用法：

-   在函数中调用t.ReportAllocs()，启用内存使用分析

<!-- -->

    # /usr/local/go/src/bufio/bufio_test.go
    func BenchmarkWriterFlush(b *testing.B) {
        b.ReportAllocs()
        bw := NewWriter(ioutil.Discard)
        str := strings.Repeat("x", 50)
        for i := 0; i < b.N; i++ {
            bw.WriteString(str)
            bw.Flush()
        }
    }

-   通过 b.StopTimer()、b.ResetTimer()、b.StartTimer()来停止、重置、启动
    时间经过和内存分配计数

<!-- -->

    # /usr/local/go/src/fmt/scan_test.go
    func BenchmarkScanInts(b *testing.B) {
        b.ResetTimer()
        ints := makeInts(intCount)
        var r RecursiveInt
        for i := b.N - 1; i >= 0; i-- {
            buf := bytes.NewBuffer(ints)
            b.StartTimer()
            scanInts(&r, buf)
            b.StopTimer()
        }
    }

-   调用b.SetBytes()记录在一个操作中处理的字节数

<!-- -->

    # /usr/local/go/src/testing/benchmark.go
    func BenchmarkFields(b *testing.B) {
        b.SetBytes(int64(len(fieldsInput)))
        for i := 0; i < b.N; i++ {
            Fields(fieldsInput)
        }
    }

-   通过b.RunParallel()方法和
    \*testing.PB类型的Next()方法来并发执行被测对象

<!-- -->

    # /usr/local/go/src/sync/atomic/value_test.go
    func BenchmarkValueRead(b *testing.B) {
        var v Value
        v.Store(new(int))
        b.RunParallel(func(pb *testing.PB) {
            for pb.Next() {
                x := v.Load().(*int)
                if *x != 0 {
                    b.Fatalf("wrong value: got %v, want 0", *x)
                }
            }
        })
    }

#### 测试辅助代码

测试辅助代码是编写测试代码过程中因代码重用和代码质量考虑而产生的。主要包括如下方面：

-   引入依赖的外部包，如每个测试文件都需要的 testing 包等：

<!-- -->

    # /usr/local/go/src/log/log_test.go:
    import (
        "bytes"
        "fmt"
        "os"
        "regexp"
        "strings"
        "testing"
        "time"
    )

-   定义多次用到的常量和变量，测试用例数据等：

<!-- -->

    # /usr/local/go/src/log/log_test.go:
    const (
        Rdate         = `[0-9][0-9][0-9][0-9]/[0-9][0-9]/[0-9][0-9]`
        Rtime         = `[0-9][0-9]:[0-9][0-9]:[0-9][0-9]`
        Rmicroseconds = `\.[0-9][0-9][0-9][0-9][0-9][0-9]`
        Rline         = `(57|59):` // must update if the calls to l.Printf / l.Print below move
        Rlongfile     = `.*/[A-Za-z0-9_\-]+\.go:` + Rline
        Rshortfile    = `[A-Za-z0-9_\-]+\.go:` + Rline
    )

    // ...

    var tests = []tester{
        // individual pieces:
        {0, "", ""},
        {0, "XXX", "XXX"},
        {Ldate, "", Rdate + " "},
        {Ltime, "", Rtime + " "},
        {Ltime | Lmicroseconds, "", Rtime + Rmicroseconds + " "},
        {Lmicroseconds, "", Rtime + Rmicroseconds + " "}, // microsec implies time
        {Llongfile, "", Rlongfile + " "},
        {Lshortfile, "", Rshortfile + " "},
        {Llongfile | Lshortfile, "", Rshortfile + " "}, // shortfile overrides longfile
        // everything at once:
        {Ldate | Ltime | Lmicroseconds | Llongfile, "XXX", "XXX" + Rdate + " " + Rtime + Rmicroseconds + " " + Rlongfile + " "},
        {Ldate | Ltime | Lmicroseconds | Lshortfile, "XXX", "XXX" + Rdate + " " + Rtime + Rmicroseconds + " " + Rshortfile + " "},
    }

-   和普通的Golang源代码一样，测试代码中也能定义init函数，init函数会在引入外部包、定义常量、声明变量之后被自动调用，可以在init函数里编写测试相关的初始化代码。

<!-- -->

    # /usr/local/go/src/bytes/buffer_test.go
    func init() {
        testBytes = make([]byte, N)
        for i := 0; i < N; i++ {
            testBytes[i] = 'a' + byte(i%26)
        }
        data = string(testBytes)
    }

-   封装测试专用的公共函数，抽象测试专用的结构体等：

<!-- -->

    # /usr/local/go/src/log/log_test.go:
    type tester struct {
        flag    int
        prefix  string
        pattern string // regexp that log output must match; we add ^ and expected_text$ always
    }

    // ...

    func testPrint(t *testing.T, flag int, prefix string, pattern string, useFormat bool) {
        // ...
    }

#### 示例函数

示例函数无需接收参数，但需要使用注释的 `Output:`
标记说明示例函数的输出值，未指定`Output:`标记或输出值为空的示例函数不会被执行。

示例函数需要归属于某个 包/函数/类型/类型 的方法，具体命名规则如下：

    func Example() { ... }      # 包的示例函数
    func ExampleF() { ... }     # 函数F的示例函数
    func ExampleT() { ... }     # 类型T的示例函数
    func ExampleT_M() { ... }   # 类型T的M方法的示例函数

    # 多示例函数 需要跟下划线加小写字母开头的后缀
    func Example_suffix() { ... }
    func ExampleF_suffix() { ... }
    func ExampleT_suffix() { ... }
    func ExampleT_M_suffix() { ... }

go doc 工具会解析示例函数的函数体作为对应 包/函数/类型/类型的方法
的用法。

测试函数的相关说明，可以通过`go help testfunc`来查看帮助文档。

### 使用 go test 工具

Golang中通过命令行工具`go test`来执行测试代码，打开shell终端，进入需要测试的包所在的目录执行
`go test`，或者直接执行`go test $pkg_name_in_gopath`即可对指定的包执行测试。

通过形如`go test github.com/tabalt/...`的命令可以执行`$GOPATH/github.com/tabalt/`目录下所有的项目的测试。`go test std`命令则可以执行Golang标准库的所有测试。

如果想查看执行了哪些测试函数及函数的执行结果，可以使用`-v`参数：

    [tabalt@localhost hello] go test -v
    === RUN   TestHello
    --- PASS: TestHello (0.00s)
    === RUN   ExampleHello
    --- PASS: ExampleHello (0.00s)
    PASS
    ok      hello  0.006s

假设我们有很多功能测试函数，但某次测试只想执行其中的某一些，可以通过-run参数，使用正则表达式来匹配要执行的功能测试函数名。如下面指定参数后，功能测试函数`TestHello`不会执行到。

    [tabalt@localhost hello] go test -v -run=xxx
    PASS
    ok      hello  0.006s

性能测试函数默认并不会执行，需要添加-bench参数，并指定匹配性能测试函数名的正则表达式；例如，想要执行某个包中所有的性能测试函数可以添加参数`-bench .`
或 `-bench=.`。

    [tabalt@localhost hello] go test -bench=.
    PASS
    BenchmarkHello-8     2000000           657 ns/op
    ok      hello  1.993s

想要查看性能测试时的内存情况，可以再添加参数`-benchmem`：

    [tabalt@localhost hello] go test -bench=. -benchmem
    PASS
    BenchmarkHello-8     2000000           666 ns/op         208 B/op          9 allocs/op
    ok      hello  2.014s

参数`-cover`可以用来查看我们编写的测试对代码的覆盖率：

    [tabalt@localhost hello] go test -cover
    PASS
    coverage: 100.0% of statements
    ok      hello  0.006s

详细的覆盖率信息，可以通过`-coverprofile`输出到文件，并使用`go tool cover`来查看，用法请参考`go tool cover -help`。

更多`go test`命令的参数及用法，可以通过`go help testflag`来查看帮助文档。

### 高级测试技术

#### IO相关测试

testing/iotest包中实现了常用的出错的Reader和Writer，可供我们在io相关的测试中使用。主要有：

-   触发数据错误dataErrReader，通过DataErrReader()函数创建

-   读取一半内容的halfReader，通过HalfReader()函数创建

-   读取一个byte的oneByteReader，通过OneByteReader()函数创建

-   触发超时错误的timeoutReader，通过TimeoutReader()函数创建

-   写入指定位数内容后停止的truncateWriter，通过TruncateWriter()函数创建

-   读取时记录日志的readLogger，通过NewReadLogger()函数创建

-   写入时记录日志的writeLogger，通过NewWriteLogger()函数创建

#### 黑盒测试

testing/quick包实现了帮助黑盒测试的实用函数 Check和CheckEqual。

Check函数的第1个参数是要测试的只返回bool值的黑盒函数f，Check会为f的每个参数设置任意值并多次调用，如果f返回false，Check函数会返回错误值
\*CheckError。Check函数的第2个参数
可以指定一个quick.Config类型的config，传nil则会默认使用quick.defaultConfig。quick.Config结构体包含了测试运行的选项。

    # /usr/local/go/src/math/big/int_test.go
    func checkMul(a, b []byte) bool {
        var x, y, z1 Int
        x.SetBytes(a)
        y.SetBytes(b)
        z1.Mul(&x, &y)

        var z2 Int
        z2.SetBytes(mulBytes(a, b))

        return z1.Cmp(&z2) == 0
    }

    func TestMul(t *testing.T) {
        if err := quick.Check(checkMul, nil); err != nil {
            t.Error(err)
        }
    }

CheckEqual函数是比较给定的两个黑盒函数是否相等，函数原型如下：

    func CheckEqual(f, g interface{}, config *Config) (err error)

#### HTTP测试

net/http/httptest包提供了HTTP相关代码的工具，我们的测试代码中可以创建一个临时的httptest.Server来测试发送HTTP请求的代码:

    ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "Hello, client")
    }))
    defer ts.Close()

    res, err := http.Get(ts.URL)
    if err != nil {
        log.Fatal(err)
    }

    greeting, err := ioutil.ReadAll(res.Body)
    res.Body.Close()
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("%s", greeting)

还可以创建一个应答的记录器httptest.ResponseRecorder来检测应答的内容：

    handler := func(w http.ResponseWriter, r *http.Request) {
        http.Error(w, "something failed", http.StatusInternalServerError)
    }

    req, err := http.NewRequest("GET", "http://example.com/foo", nil)
    if err != nil {
        log.Fatal(err)
    }

    w := httptest.NewRecorder()
    handler(w, req)

    fmt.Printf("%d - %s", w.Code, w.Body.String())

#### 测试进程操作行为

当我们被测函数有操作进程的行为，可以将被测程序作为一个子进程执行测试。下面是一个例子：

    //被测试的进程退出函数
    func Crasher() {
        fmt.Println("Going down in flames!")
        os.Exit(1)
    }

    //测试进程退出函数的测试函数
    func TestCrasher(t *testing.T) {
        if os.Getenv("BE_CRASHER") == "1" {
            Crasher()
            return
        }
        cmd := exec.Command(os.Args[0], "-test.run=TestCrasher")
        cmd.Env = append(os.Environ(), "BE_CRASHER=1")
        err := cmd.Run()
        if e, ok := err.(*exec.ExitError); ok && !e.Success() {
            return
        }
        t.Fatalf("process ran with err %v, want exit status 1", err)
    }

### 参考资料

[https://talks.golang.org/2014...](https://talks.golang.org/2014/testing.slide#11)\
<https://golang.org/pkg/testing/>\
[https://golang.org/pkg/testin...](https://golang.org/pkg/testing/iotest/)\
[https://golang.org/pkg/testin...](https://golang.org/pkg/testing/quick/)\
[https://golang.org/pkg/net/ht...](https://golang.org/pkg/net/http/httptest/)

原文链接：[http://tabalt.net/blog/golang...](http://tabalt.net/blog/golang-testing/)

