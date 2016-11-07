---
date: 2016-03-08T10:44:36+08:00
description: ""
disqus_identifier: "20160301"
slug: "understanding-golang-import-package"
tags:
- Go
- import
- package
title: 理解Go import
topics:
- 开发
---

Go 的源代码是按 package 方式组织，再通过 import 引入使用。在理解 improt 前有必要来罗嗦下 Go 的代码组织结构 和理解 package。

### 工作目录

在 Go 中代码保持在称之为 workspace 的系统文件夹中。这个工作目录下有三个根目录：

+ __bin__ 	包含可执行文件	。
+ __pkg__ 	包含不同操作系统架构的包二进制文件。相当于编译后的库文件。
+ __src__ 	包含按包方式组织的源代码。

其中 bin 和 pkg 文件夹是在调用 go 命令安装和编译源代码时自动生成。而 src 下可组织多个包，且能使用版本控制工具。

注意，必须让 Go 知道工作目录的位置，这样才能知道包的具体位置。通过设置环境变量 `GOPATH` 来指定。
```bash
export GOPATH=/home/dev/gowork/ 
```

### 导入包

improt 便是用一个独一无二的字符串路径来指向包，而包的导入路径是基于工作目录或者的。因为 Go 会在 工作目录 src 子目录下查找包。

标准包使用的是给定的短路径，如“fmt”、“net/http”，但你自己的包，需要在工作目录下指定一个目录，同时预防和以后的代码包路径发生冲突。以便我们作为个人开发者，基于 github 的话，那么我们可以建立自己的代码基目录，`github.com/user` 。


improt 则导入包，既然是包地址，实际上就是基于工作目录的文件夹目录。如：

先创建代码库: $GOPATH/src/ysqi/lib/lib.go
```Go
package lib

import "fmt"

func SayHello() {
	fmt.Println("Hello,YSQ :) ")
}
```

再新建一个App：$GOPATH/src/ysqi/app/main.go
```Go
package main

import "ysqi/lib"

func main() {
	lib.SayHello()
}

```
如上，使用`import "ysqi/lib"` 导入包，从而调用 SysHello 方法 。再看看 这个代码`lib.SyaHello`中的`lib`标识的含义。他标示调用包`lib`，而对比 import "ysqi/lib" 中的 `lib`，此 lib 是包路径的一部分，并不代表包名。

也就是说 `ysqi/lib` 和 `lib.SyyHello()` 里的`lib`含义不同。包名和包路径不需要完全一致，如：
```Go
# file Path: $GOPATH/src/ysqi/mylib/lib.go

package lib

import "fmt"

func SayHello() {
	fmt.Println("Hello,I'm in myLib :) ")
}
```
调用 lib 的方式变成了`import "ysqi/mylib"`：
```Go
package main

import "ysqi/mylib"

func main() {
	lib.SayHello()
}
```

那么 一个目录下可以有多个包吗？ 这是不可以的，Go 编译失败，错误信息：
```bash
can't load package: package .: found packages lib (lib.go) and mylib (mylib.go) in ...
```

### 导入包的多种方式

实际上，导入使用包有三种方式，不同方式用途不一样：

+ import   "ysqi/lib"   

常规方式，通过包名`lib`调用`SayHello`方法。lib.SayHello()

+ import m "ysqi/lib" 

这里的 __m__ 是包名 lib 的别名，m.SyayHello() 。该方式合适的场景：

1.包名过于复杂或者意思不明确。

如使用 `mywebapp/libs/mongodb/db`  包时，不确定此 db 是哪种类型，故可以使用别名来明确含义：
```Go
import mongo "mywebapp/libs/mongodb/db"
```

2.包名和其他包冲突。

世界之大，变化无穷。现在我们有库 db ,但没过几年出现了另一种DB，叫云DB。但包名是一样的，分别用别名区分：
```Go
import mongo "mywebapp/libs/mongodb/db"
import ydbgo "mywebapp/libs/yundb/db"
```


+ import . "ysqi/lib"

这里的点`.`符号表示，对包 lib 的调用直接省略包名，你我以后就是一家人，不分彼此，你的东西就像我就的一样，随便用。
```Go
package main

import . "ysqi/lib"

func main() {
	SayHello()
}
```

+ improt _ "ysqi/lib"

这里说的是我还不准备现在使用你们家的东西，但得提前告诉一声。你先做好准备，先准备好饭菜，等我来就行，也有可能我压根就不来。
```Go
package main

import _ "ysqi/lib"

func main() {
	 
}
```

特殊符号__“_”__ 仅仅会导致 lib 执行初始化工作，如初始化全局变量，调用 init 函数。


### 总结

import 导入 Go 包有几种方式，用途不同。 代码统一存储在工作目录下，工作目录里边有会很多个包，不同包按目录组织，包下面由多个代码文件组成。导入包时按包的唯一路径进行导入，导入的包默认是必须要使用，如果不使用则编译失败，需要移除，减少不必要代码的引入，当然还有其他使用场景。默认情况下，我们使用文件名做为包名，方便理解。不同包组织不同的功能实现，方便理解。 