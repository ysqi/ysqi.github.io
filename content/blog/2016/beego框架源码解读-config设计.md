---
date: 2016-02-13T08:04:49+08:00
description: ""
disqus_identifier: "20160003"
slug: "beego-webframework-config"
tags:
- Go
- beego
title: beego框架源码解读 config 模块设计
topics:
- 开发
---

这是关于 beego 的第一篇正式的源代码解读文章，前面文章 [beego框架源码解读计划] 中已介绍解读计划。而本文主要针对 beego 框架 config 模块展开讨论。通过本文你可掌握 beego config 使用。

### config 简要说明

beego 的各个模块设计基本相同，灵感源于 Go 内部 database/sql 包的 [sql库驱动注册] 。模块实现后进行注册，使用时通过注册名进行访问初始化。
而 config 模块也相同。分别支持 ini、json、xml、yaml、内存 格式的配置文件管理。使用时调用`config.NewConfig("ini", "config path")`来构建不同格式 config 解析器。

### 初始化 config 解析器

首先需确认所用的配置文件格式，beego 的 appconfig 默认使用 ini。 再通过类型初始化配置文件解析器对象。
```go
import (
	"github.com/astaxie/beego/config"
)

func main() {
	cfg, err := config.NewConfig("ini", "myconfig.ini")
	if err != nil {
		// error
	}
}
```
使用 config 模块时需要导入`github.com/astaxie/beego/config`包 ，再调用`config.NewConfig`来指定文件类型，并加载解析 `myconfig.ini` 配置文件。当加载解析失败时，会返回错误信息，故不能忽略该错误信息。

当然，也可以直接解析 []byte 数据，构建解析器。这样做法不常见，有时能用于解析配置片断。
```go
 inicontext = `
app = app
;comment one
#comment two
# comment three
appname = beeapi
httpport = 8080
# DB Info
# enable db
[dbinfo]
# db type name
# suport mysql,sqlserver
name = mysql
`
cfg, err := config.NewConfigData("ini", []byte(inicontext))
``` 

上面仅仅是按照 ini 提供的示例，config 模块还内置支持 json、xml、yaml 格式。但在使用 xml 或者 yaml 时需要多两个操作。

1.需要更新获取所依赖的第三方包：`github.com/beego/x2j`

```bash
go get -u github.com/astaxie/beego/config/xml
```
2.为了向 config 模块注册 xml 解析器，需要在`config.NewConfigData("xml","data.xml" )` 前导入包进行初始化。

```go
import _ "github.com/astaxie/beego/config/xml"。
```

### 使用 config 解析器

前面使用已构建解析器，后续则可以使用 config 所提供的一系列方法操作配置文件内容。

+ Set(key, val string) error //设置指定 Key 的 Value，支持多级模式，如： section::key 形式。
+ String(key string) string //获取 KEY 对应的 Value，支持多级模式
+ Strings(key string) []string //获取 KEY 对应的 Value 组数据
+ Int(key string) (int, error) //获取 Int 类型的的 Value
+ Int64(key string) (int64, error) //获取 Int64 类型的 Value
+ Bool(key string) (bool, error) //获取 Bool 类型的 Value
+ Float(key string) (float64, error) //获取 Float64 类型的 Value
+ DefaultString(key string, defaultVal string) string  
+ DefaultStrings(key string, defaultVal []string) []string
+ DefaultInt(key string, defaultVal int) int
+ DefaultInt64(key string, defaultVal int64) int64
+ DefaultBool(key string, defaultVal bool) bool
+ DefaultFloat(key string, defaultVal float64) float64
+ DIY(key string) (interface{}, error) //获取任意类型数据
+ GetSection(section string) (map[string]string, error) //获取节点下数据
+ SaveConfigFile(filename string) error //保存配置文件到指定文件

其中 `Strings(key string) []string` 方法是将 String 值通过`;`进行分割。而 `GetSection`则是返回一个节点下所有的数据，此方法方便在使用一组配置信息时调用。如对于数据库连接信息均配置在`[db]`节点下，则在初始化数据库信息时则可直接获取组信息。当然如果是有默认值得信息，则可以通过带默认的方式获取，如`cfg.DefaultString("httpprot","8080")`获取 http port 信息，如果未设置则使用默认值 8080。

### config 解析器特性

config 在解析配置文件时便将文件信息存储在 map 中，故不会实时从配置文件读取数据，则能保证高效，其实配置文件也不需要监听变化，在程序运行之初加载到内存即可。

另外, config 解析器支持 `include` 模式，即解析多个配置文件。这非常适合不同配置信息的分离，或者生产配置和开发配置的分割。

其次，在 beego 的 1.6.0的下一个版本中将支持配置文件实时读取环境变量信息。这能有效保护敏感信息，如ssh key、password 等。

当然，你可以自定义解析器，只需要实现接口，并注册到 config 模块即可。具体可以参加 xml 解析器的实现。

### config include 的使用

这是非常棒的功能，如同 Nginx.config 配置，支持 include ，但目前仅能在 `ini` 配置文件中使用。
```ini
http {
     #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
}
```
使用 include 的好处之一是当配置信息过多时，能将配置信息分散到不同文件中。如:
```ini
appname = demo
httpport = 8080
runmode = dev

include web.conf
include cache.conf
```
include 文件默认从 app/config/ 目录下查找。include 可以覆盖前面的配置信息，但是要注意如果使用 include 则不应该通过 `SaveConfigFile` 再保存配置信息。

上面就是讲的 config 模块，从上也知道，注意是讲究的 ini 格式配置文件。实际上在程序中使用场景的 ini 已经能满足基本要求。大家可以将 config 模块集成到你自己的程序中使用。

希望本文能帮助你更多的了解使用 beego 的 config 模块。接下来会讲 beego 框架下 config 的使用。另外 beego config 模块官方介绍文档见:http://beego.me/docs/module/config.md ,godoc 文档见：https://godoc.org/github.com/astaxie/beego/config 。

[beego框架源码解读计划]: {{< ref "Beego框架源码分析计划.md" >}}
[sql库驱动注册]: https://golang.org/src/database/sql/sql.go?s=805:853#L24

