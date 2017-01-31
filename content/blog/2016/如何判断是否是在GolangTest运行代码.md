---
date: 2016-10-03T19:51:14+08:00
description: ""
disqus_identifier: "20161003"
slug: "how-to-know-running-within-gotest"
tags:
- Go
- Go Test
title: 如何判断是否是在GolangTest运行代码
topics:
- 编程语言与开发
---

在进行Go Test时需要对代码进行特殊初始化,但是如何才能在运行Go Test时就能知晓代码是运行在Go Test模式下的呢?

经过各种尝试,总算能非常靠谱的进行Go Test 识别! 

### 需求
在运行Go Test时需要和 Go 调式区分配置文件,Test下使用特定配置文件,已区别其他情况下的数据库等配置信息.
如:
+ 在 go run 下使用的配置信息
```INI
APP = DEV开发
port = 8080
dbName = DevATOP
```

+ 在 go test 下使用的配置信息

```INI
APP = JUSTTest
port = 6060
dbName = TESTATOP
```

### 解决方案

在初始化配置时,判断所属模式.不同模式使用不同配置文件:
+ go run 模式下通过文件 app.dev.cfg 初始化配置
+ go test 模式下通过文件 app.test.cfg 初始化配置

### 代码实现

如何运行时判断模式,一般有几种做饭,但即便上就是需要一个模式标记


1. 运行时携带参数 `-runmode=test|dev`的方式,解析运行参数来加载
```Go 
func init() {
    runMode := flag.String("runMode","dev","代码运行模式") 
    flag.Parse()
      
    cfg :="app.dev.cfg"
    if *runMode="test"{
        cfg="app.test.cfg"
    } 
}
```
此方式不太适运用在 go test 下,因为你比较运行 go test 时基本上基于工具,而非手工执行命令,不方便添加入参

2. 运行时直接携带参数 `-config=/app/app.dev.cfg` 
```Go  
func init() {
    cfg := flag.String("config","./app.dev.cfg","配置文件路径") 
    flag.Parse() 
}
```
这种方式虽然灵活,但还是同上面一种方式运用在 go test 下

3. 使用环境变量,形式同方案2
```Go 
func init() {
    cfg := os.Getenv("appConfigPATH") 
}
```

### 最佳方案

不是说上面的方式不好,而是运用到 go test 下,便不那么好使,特别是我现在使用的是VS Code. 下面这种方式变为 Go test 而生了.

基于Go test 工具带有多个参数,参数Flag有默认值,在 test 运行时进行解析.

具体参数可见: `go test -help` .

既然有参数,则我们组需要去判断运行时是否包含 go test 所携带的参数,如:
> `-v` 
>
> Verbose output: log all tests as they are run. Also print all
text from Log and Logf calls even if the test succeeds.

因此在代码中的判断如下:
```Go
func init() {
    cfg :="app.dev.cfg"
    if flag.Lookup("test.v") != nil {
        //flag不为空,则说明存在test所拥有的参数,是在 go test 模式
        cfg="app.test.cfg"
    }
}
```

此外完整代码,打印下所有的参数,实际上就是 `go test -help`的参数信息:
```Go
package main

import (
	"flag"
	"fmt"
)

func main() {
	sayHello()
}

func sayHello() { 
	flag.VisitAll(func(f *flag.Flag) {
		fmt.Printf("%#v \n", f)
	}) 
}
```
```Go
package main

import "testing"

func TestSayHello(t *testing.T) {
	sayHello()
}

```
Output:
```Bash
=== RUN   TestSayHello
print Flag...
&flag.Flag{Name:"test.bench", Usage:"regular expression per path component to select benchmarks to run", Value:(*flag.stringValue)(0xc420010390), DefValue:""} 
&flag.Flag{Name:"test.benchmem", Usage:"print memory allocations for benchmarks", Value:(*flag.boolValue)(0xc4200103b2), DefValue:"false"} 
&flag.Flag{Name:"test.benchtime", Usage:"approximate run time for each benchmark", Value:(*flag.durationValue)(0xc420010288), DefValue:"1s"} 
&flag.Flag{Name:"test.blockprofile", Usage:"write a goroutine blocking profile to the named file after execution", Value:(*flag.stringValue)(0xc4200104b0), DefValue:""} 
&flag.Flag{Name:"test.blockprofilerate", Usage:"if >= 0, calls runtime.SetBlockProfileRate()", Value:(*flag.intValue)(0xc420010488), DefValue:"1"} 
&flag.Flag{Name:"test.count", Usage:"run tests and benchmarks `n` times", Value:(*flag.uintValue)(0xc4200103f8), DefValue:"1"} 
&flag.Flag{Name:"test.coverprofile", Usage:"write a coverage profile to the named file after execution", Value:(*flag.stringValue)(0xc420010410), DefValue:""} 
&flag.Flag{Name:"test.cpu", Usage:"comma-separated list of number of CPUs to use for each test", Value:(*flag.stringValue)(0xc420010510), DefValue:""} 
&flag.Flag{Name:"test.cpuprofile", Usage:"write a cpu profile to the named file during execution", Value:(*flag.stringValue)(0xc420010490), DefValue:""} 
&flag.Flag{Name:"test.memprofile", Usage:"write a memory profile to the named file after execution", Value:(*flag.stringValue)(0xc420010450), DefValue:""} 
&flag.Flag{Name:"test.memprofilerate", Usage:"if >=0, sets runtime.MemProfileRate", Value:(*flag.intValue)(0xc420010470), DefValue:"0"} 
&flag.Flag{Name:"test.outputdir", Usage:"directory in which to write profiles", Value:(*flag.stringValue)(0xc4200103d0), DefValue:""} 
&flag.Flag{Name:"test.parallel", Usage:"maximum test parallelism", Value:(*flag.intValue)(0xc420010508), DefValue:"8"} 
&flag.Flag{Name:"test.run", Usage:"regular expression to select tests and examples to run", Value:(*flag.stringValue)(0xc420010430), DefValue:""} 
&flag.Flag{Name:"test.short", Usage:"run smaller test suite to save time", Value:(*flag.boolValue)(0xc4200103c5), DefValue:"false"} 
&flag.Flag{Name:"test.timeout", Usage:"if positive, sets an aggregate time limit for all tests", Value:(*flag.durationValue)(0xc420010500), DefValue:"0s"} 
&flag.Flag{Name:"test.trace", Usage:"write an execution trace to the named file after execution", Value:(*flag.stringValue)(0xc4200104e0), DefValue:""} 
&flag.Flag{Name:"test.v", Usage:"verbose: print additional output", Value:(*flag.boolValue)(0xc4200103cc), DefValue:"false"} 
print end.
--- PASS: TestSayHello (0.00s)
PASS
ok  	github.com/ysqi/try	0.006s
Success: Tests passed.
```


### 总结

采用一种巧妙的方式,便能清晰的标识出 go test 场景,无入侵,简洁清晰. 其实此方式也是在 Stackoverflow 看到的:
http://stackoverflow.com/questions/14249217/how-do-i-know-im-running-within-go-test ,在此一记,给大家提供一种思路,如何在运行时判断是否是 go test 运行模式.
