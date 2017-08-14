---
title: "Golang Generate"
date: 2017-08-14T18:52:10+08:00
draft: false 
description: ""
slug: "go-command-generate" 
tags:
- "Go"
- "command"
topics: 
- 编程语言与开发
---


前期有专门利用`go generate`自动生成Go代码，今日在查看Go源代码时发现有大量使用此命令已生成各类代码。故在此特写文章说明`generate`命令的神奇之处。


## 命令诉求
通用计算有一特性——[图灵完备][tuling]。是一个计算机程序能编写一个计算机程序。既能写程序的程序。按规则定义描述内容，则可以根据描述生成程序代码。10年时刚做项目便以增删改查为主，代码生成器生成代码那是杠杠的。

通过定义便可高效生成代码，无需手工编码。如当定义一个枚举后，为了打印友好内容，我们经常手工定义String方法。

```go
type Status int

const (
	Offline Status = iota
	Online
	Disable
	Deleted
)

var statusText = []string{"Offline", "Online", "Desable", "Deleted"}

func (s Status) String() string {
	v := int(s)
	if v < 0 || v > len(statusText) {
		return fmt.Sprintf("Status(%d)", s)
	}
	return statusText[v]
}

```
当遇到枚举调整时，则必须要再同步修改`statusText`，而此事常容被忽视。




## Generate命令说明
早在Go1.4版本实现，所以你现在可以看到Go源码中大量含有的该命令使用。

如：在unicode包中生产Unicode表，为encoding/gob创建有效的编解码方法，在time包中创建时区数据等等

**go generate**用于一键式批量执行**任何**命令，创建或更新Go文件或者输出结果。

Generate 命令和其他go build、go get、go test等没半毛钱关系。需特定执行，命令如下：
```shell
go generate [-run regexp] [-n] [-v] [-x] [build flags] [file.go... | packages]
```
参数说明：

+ -run 正则表达式匹配命令行，仅执行匹配的命令
+ -v 打印已被检索处理的文件。
+ -n 打印出将被执行的命令，此时将不真实执行命令
+ -x 打印已执行的命令

执行举例：
```shell
# 打印当前目录下所有文件，将被执行的命令
go generate -n ./...
# 对包下所有Go文件进行处理
go generate github.com/ysqi/repo
# 打印包下所有文件，将被执行的命令
go generate -n runtime
```

## 如何使用Generate命令
需在的代码中配置generate标记，则在执行`go generate`时可被检测到。`go generate`执行时，实际在扫描如下内容：
```go
//go:generate command argument...
```
generate命令不是解析文件，而是逐行匹配以**//go:generate** 开头的行(前面不要有空格)。故命令可以写在任何位置，也可存在多个命令行。

**//go:generate** 后跟随具体的命令。命令为可执行程序，形同在Shell下执行。所以命令是在环境变量中存在，也可是完整路径。如：
```go
package main
import "fmt"
//go:generate echo hello
//go:generate go run main.go
//go:generate  echo file=$GOFILE pkg=$GOPACKAGE
func main() {
	fmt.Println("main func")
}
```
执行：
```shell
$ go generate
hello
man func
file=main.go pkg=main
```

在执行`go generate`时将会加入些信息到环境变量，可在命令程序中使用。
```text
$GOARCH
	架构 (arm, amd64, etc.)
$GOOS
	OS (linux, windows, etc.)
$GOFILE
	当前处理中的文件名
$GOLINE
	当前命令在文件中的行号
$GOPACKAGE
    当前处理文件的包名
$DOLLAR
	固定的"$",不清楚用途
```

同时该命令是在所处理文件的目录下执行，故利用上述信息和当前目录，边可获得丰富的DIY功能。


## Generate实战
如前面所述，现在练手来实现对枚举String函数以打印友好信息。 最终调用为：
```
//go:generate  myenumstr -type Status
```
自动生成如下代码：
```go
package user

import "fmt"

func (c Status) String() string {
	switch c {
	case Offline:
		return "Offline"
	case Online:
		return "Online"
	case Disable:
		return "Disable"
	case Deleted:
		return "Deleted"
	}
	return fmt.Sprintf("Status(%d)", c)
}
```

### 业务逻辑
该如何实现呢？实际我们需要获得三项:

+ 包名：user，该文件将存放在当前目录，需要知晓包名称
+ 类型名：Status，参数传递
+ 所有同类型var:  Offline，Online，Disable，Deleted

再生成代码后将其保存到当前目录，同时进行gofmt。

### 具体实现

1.通过环境变量获取包名称
```go
pkgName = os.Getenv("GOPACKAGE")
```
2.获取当前目录包信息
这里利用Go内置的库`go/build`解析目录，则可以获得该文件夹下包信息。
```go
var err error
pkgInfo, err = build.ImportDir(".", 0)
if err != nil {
	log.Fatal(err)
}
```
至此便能获得目录下所有Go文件`pkgInfo.GoFiles`，用于语法树解析。

3.解析Go文件语法树，提取Status相关信息。
需要注意的是，我们约定所定义的枚举信息实际应该全部是Const。需从语法树中
提取出所有的Const，并判断类型是否符合条件。

这里利用的是Go的语法树库[go/ast][ask](abstract syntax tree)和解析库[go/parser][parser]，语法树是按语句块`()`形成树结构。从中过滤`const`语句块
```go
fset := token.NewFileSet()
//解析go文件
f, err := parser.ParseFile(fset, gofile, nil, 0)
if err != nil {
	log.Fatal(err)
}
//遍历每个树节点
ast.Inspect(f, func(n ast.Node) bool {
	decl, ok := n.(*ast.GenDecl)
	if !ok || decl.Tok != token.CONST {
		return true
	}
	//...
}
```
再循环const语句块中最小部分：
```go
for _, spec := range decl.Specs {}
``` 
对每小部分判断，并获得对应的Type，如果Type为空则同上一行的一致。
```go
// Const 代码块
const (
	Offline Status = iota
	Online
	Disable
	Deleted
)
```
行内容
```go
vspec := spec.(*ast.ValueSpec) 
if vspec.Type == nil && len(vspec.Values) >0 {
	// 排除 v = 1 这种结构
	typ = ""
	continue
}
//如果Type不为空，则确认typ
if vspec.Type != nil {
	ident, ok := vspec.Type.(*ast.Ident)
	if !ok {
		continue
	}
	typ = ident.Name
}
```
但我们只需要获得符合要求的Type，获得符合要求的Const信息：
```go
consts, ok := typesMap[typ]
if !ok {
	continue
}
for _, n := range vspec.Names {
	consts = append(consts, n.Name)
}
typesMap[typ] = consts
```
这里的typesMap是根据程序入参建立：
```go
var ( 
	typeNames = flag.String("type", "", "")
)
types := strings.Split(*typeNames, ",")
typesMap := make(map[string][]string, len(types))
for _, v := range types {
	typesMap[strings.TrimSpace(v)] = []string{}
}
```

4.根据收集的Const，生成String函数

使用模板生成内容，同时进行gofmt
```go
func genString(types map[string][]string) []byte {

	const strTmp = `
	package {{.pkg}}
	import "fmt"
	
	{{range $typ,$consts :=.types}}
	func (c {{$typ}}) String() string{
		switch c { {{range $consts}}
			case {{.}}:return "{{.}}"{{end}}
		}
		return fmt.Sprintf("Status(%d)", c)	
	}
	{{end}}
	`
	pkgName := os.Getenv("GOPACKAGE")
	if pkgName == "" {
		pkgName = pkgInfo.Name
	}
	data := map[string]interface{}{
		"pkg":   pkgName,
		"types": types,
	}

	//利用模板库，生成代码文件
	t, err := template.New("").Parse(strTmp)
	if err != nil {
		log.Fatal(err)
	}
	buff := bytes.NewBufferString("")
	err = t.Execute(buff, data)
	if err != nil {
		log.Fatal(err)
	}
	//进行格式化
	src, err := format.Source(buff.Bytes())
	if err != nil {
		log.Fatal(err)
	}
	return src
}
```

5.保存代码到文件
将文件直接保存到当前目录下，文件名已"_string"结尾
```go
src := genString(consts) 

outputName := ""
if outputName == "" {
	types := strings.Split(*typeNames, ",")
	baseName := fmt.Sprintf("%s_string.go", types[0])
	outputName = filepath.Join(".", strings.ToLower(baseName))
}
err := ioutil.WriteFile(outputName, src, 0644)
if err != nil {
	log.Fatalf(err)
}
```

6.在status.go源文件配置标记
```go
//go:generate myenumstr -type Status
```
运行`go generate`，出现错误：
```text
user/status.go:5: running "myenumstr": exec: "myenumstr": executable file not found in $PATH
```
此时需要将myenumstr.go install
```shell
go install github.com/ysqi/string/myenumstr.go
```
安装后，我们可以在status.go使用两种方式调用：
```go
//go:generate myenumstr -type Status
//go:generate go run github.com/ysqi/string/myenumstr.go -type Status
```

至此写完，完整代码如下：
#### 源代码-user.go
```go 
package user

type Status int

//go:generate myenumstr -type Status,Color
const (
	Offline Status = iota
	Online
	Disable
	Deleted
)

type Color int

const (
	Write Color = iota
	Red
	Blue
)
```
#### 源代码-myenumstr.go
```go
package main

import (
	"bytes"
	"flag"
	"fmt"
	"go/ast"
	"go/build"
	"go/format"
	"go/parser"
	"go/token"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
	"strings"
	"text/template"
)

var (
	pkgInfo *build.Package
)
var (
	typeNames = flag.String("type", "", "必填，逗号连接的多个Type名")
)

func main() {
	flag.Parse()

	if len(*typeNames) == 0 {
		log.Fatal("-type 必填")
	}
	consts := getConsts()
	src := genString(consts)

	//保存到文件
	outputName := ""
	if outputName == "" {
		types := strings.Split(*typeNames, ",")
		baseName := fmt.Sprintf("%s_string.go", types[0])
		outputName = filepath.Join(".", strings.ToLower(baseName))
	}
	err := ioutil.WriteFile(outputName, src, 0644)
	if err != nil {
		log.Fatalf("writing output: %s", err)
	}

}

func getConsts() map[string][]string {
	//获得待处理的Type
	types := strings.Split(*typeNames, ",")
	typesMap := make(map[string][]string, len(types))
	for _, v := range types {
		typesMap[strings.TrimSpace(v)] = []string{}
	}

	//解析当前目录下包信息
	var err error
	pkgInfo, err = build.ImportDir(".", 0)
	if err != nil {
		log.Fatal(err)
	}

	fset := token.NewFileSet()
	for _, file := range pkgInfo.GoFiles {
		f, err := parser.ParseFile(fset, file, nil, 0)
		if err != nil {
			log.Fatal(err)
		}
		typ := ""
		ast.Inspect(f, func(n ast.Node) bool {
			decl, ok := n.(*ast.GenDecl)
			// 只需要const
			if !ok || decl.Tok != token.CONST {
				return true
			}
			for _, spec := range decl.Specs {
				vspec := spec.(*ast.ValueSpec)
				if vspec.Type == nil && len(vspec.Values) > 0 {
					// 排除 v = 1 这种结构
					typ = ""
					continue
				}
				//如果Type不为空，则确认typ
				if vspec.Type != nil {
					ident, ok := vspec.Type.(*ast.Ident)
					if !ok {
						continue
					}
					typ = ident.Name
				}
				//typ是否是需处理的类型
				consts, ok := typesMap[typ]
				if !ok {
					continue
				}
				//将所有const变量名保存
				for _, n := range vspec.Names {
					consts = append(consts, n.Name)
				}
				typesMap[typ] = consts
			}
			return true
		})
	}
	return typesMap
}

func genString(types map[string][]string) []byte {

	const strTmp = `
	package {{.pkg}}
	import "fmt"
	
	{{range $typ,$consts :=.types}}
	func (c {{$typ}}) String() string{
		switch c { {{range $consts}}
			case {{.}}:return "{{.}}"{{end}}
		}
		return fmt.Sprintf("Status(%d)", c)	
	}
	{{end}}
	`
	pkgName := os.Getenv("GOPACKAGE")
	if pkgName == "" {
		pkgName = pkgInfo.Name
	}
	data := map[string]interface{}{
		"pkg":   pkgName,
		"types": types,
	}

	//利用模板库，生成代码文件
	t, err := template.New("").Parse(strTmp)
	if err != nil {
		log.Fatal(err)
	}
	buff := bytes.NewBufferString("")
	err = t.Execute(buff, data)
	if err != nil {
		log.Fatal(err)
	}
	//格式化
	src, err := format.Source(buff.Bytes())
	if err != nil {
		log.Fatal(err)
	}
	return src
}
```

实际上Go官方已实现该功能，查看具体[stringer文档][stringer]


[tuling]: https://baike.baidu.com/item/图灵完备 
[ask]: https://golang.org/pkg/go/ast
[parser]:  https://golang.org/pkg/go/parser
[stringer]: https://godoc.org/golang.org/x/tools/cmd/stringer