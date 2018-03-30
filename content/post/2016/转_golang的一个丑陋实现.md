
---
date: 2016-12-31T11:33:30+08:00
title: "golang的一个丑陋实现"
description: ""
disqus_identifier: 1485833610561535875
slug: "golangde-yi-ge-chou-lou-shi-xian"
source: "https://segmentfault.com/a/1190000005600253"
tags: 
- golang 
topics:
- 编程语言与开发
---

好多些天前，我在`golang实践群`里问了下面代码的问题：

    package main

    import "fmt"

    type Aer interface{
        Name()string
        PrintName()
    }

    type A struct {
    }

    func (a *A) Name() string {
        return "a"
    }

    func (a *A) PrintName() {
        fmt.Println(a.Name())
    }

    type B struct {
        A
    }

    func (b *B) Name() string {
        return "b"
    }

    func getAer() Aer {
        return &B{}
    }

    func main() {
        a := getAer()
        a.printName()
    }

这个实现中，golang输出的是a，这个实现违反了通常C++，Java，Python中会输出b的实现，由于上述几个语言的思维习惯已经形成，那么这个实现就会导致很多意想不到的事情。

昨儿个在`golang实践群`中，有赞的兄弟（这位兄弟知道我提的上面的问题，并说这个是golang的实现方式）就问到了，在UnmarshalJSON的时候，为何Test字段没有被赋值，并在golang中提了[issue](https://github.com/golang/go/issues/15890)

他的代码如下：

    package main

    import (
        "encoding/json"
        "fmt"
    )

    type request struct {
        Operations map[string]op `json:"operations"`
    }
    type op struct {
      operation 
      Test string  `json:"test"`
    }
    type operation struct {
        Width  int    `json:"width"`
        Height int    `json:"height"`
    }

    func (o *operation) UnmarshalJSON(b []byte) error {
        type xoperation operation
        xo := &xoperation{Width: 500, Height: 500}
        if err := json.Unmarshal(b, xo); err != nil {
            return err
        }
        *o = operation(*xo)
        return nil
    }

    func main() {
        jsonStr := `{
                "operations": {
                    "001": {
                         "test":"test",
                        "width": 100
                    }
                }
            }`
        req := request{}
        json.Unmarshal([]byte(jsonStr), &req)
        fmt.Println(req)
    }

这个问题的本质和我提出的那个问题一样，因为op中嵌入了operation，所以有了UnmarshalJSON,符合了json包中Unmarshaler接口，所以内部用接口去处理的时候，op是满足的，但实际处理的是operation，也就是以operation作为实体来进行UnmarshalJSON，导致了诡异的错误信息。

我以为，这是golang实现中非常丑陋的一个地方。

按照耗子哥说的，如果语言实现规则是知道的，还是容易犯错误的，那就是一个坑。

这个golang的坑，估计以后还得填。

