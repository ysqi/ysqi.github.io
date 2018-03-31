
---
date: 2016-12-31T11:32:38+08:00
title: "Go语言单元测试"
description: ""
disqus_identifier: 1485833558580423533
slug: "Goyu-yan-chan-yuan-ce-shi"
source: "https://segmentfault.com/a/1190000007951944"
tags: 
- golang 
categories:
- 编程语言与开发
---

简介
----

Go 语言在设计之初就考虑到了代码的可测试性。一方面 Go 本身提供了
[testing](https://golang.org/pkg/testing/) 库，使用方法很简单;\
另一方面 go 的 package
提供了很多编译选项，代码和业务逻辑代码很容易解耦，可读性比较强（不妨对比一下C++测试框架）。
本文中，我们讨论的重点是 Go 语言中\
的单元测试，而且只讨论一些基本的测试方法，包括下面几个方面：

1.  写一个简单的测试用例

2.  Table driven test

3.  使用辅助测试函数（test helper）

4.  临时文件

这里我们只涉及到一些通用的测试方法。关于 HTTP server/client
测试，这里不做深入讨论。

阅读建议
--------

**Testing shows the presence, not the absence of bugs** -- [Edsger W.
Dijkstra](https://en.wikiquote.org/wiki/Edsger_W._Dijkstra)

在阅读本文之前，建议您对 Go 语言的 package
有一定的了解，并在实际项目中使用过，下面是一些基本的要求：

1.  了解如何在项目中 import 一个外部的package

2.  了解如何将自己的项目按照功能模块划分 package

3.  了解 struct、struct字段、函数、变量名字首字母大小写的含义（非必需）

4.  了解一些 Go语言的编译选项，比如 +build !windows（非必需）

如果你对 1、2都不太了解，建议阅读一下这篇文章[How to Write Go
Code](https://golang.org/doc/code.html,)，动手实践一下。

写一个简单的测试用例
--------------------

为了便于理解，我们首先给出一个代码片段（如果你已经使用过go
的单元测试，可以跳过这个环节）：

    // demo/equal.go
    package demo

    // a function to check if two numbers equals to each other.
    func equal(a, b int) bool {
      return a == b
    }

    // demo/equal_test.go
    package demo
    import (
      "testing"
    )

    func TestEqual(t *testing.T) {
      a := 1
      b := 1
      shouldBe := true
      if real := equal(a, b); real == shouldBe {
        t.Errorf("equal(%d, %d) should be %v, but is:%v\n", a, b, shouldBe, real)
      }
    }

上面这个例子中，如果你从来没有使用过单元测试，建议在本地开发环境中运行一次。这里有几点需要注意一下：

1.  这两个文件的父目录必须与包名一致（这里是 demo），且包名必须是在
    \$GOPATH 下

2.  测试用例的函数命名必须符合 TestXXX 格式，并且参数是 t \*testing.T

3.  了解一下 t.Errorf 与 t.Fatalf 的行为差异

Table Driven Test
-----------------

上面的测试用例中，我们一次只能测试一种情况，如果我们希望在一个 TestXXX
函数中进行很多项测试，Table Driven Test 就派上了用场。\
举个例子，假设我们实现了自己的 [Sqrt](https://golang.org/pkg/math/#Sqrt)
函数 mymath.Sqrt，我们需要对其进行测试：

首先，我们需要考虑一些特殊情况：

1.  Sqrt(+Inf) = +Inf

2.  Sqrt(±0) = ±0

3.  Sqrt(x &lt; 0) = NaN

4.  Sqrt(NaN) = NaN

然后，我们需要考虑一般情况：

1.  Sqrt(1.0) = 1.0

2.  Sqrt(4.0) = 2.0

3.  ...

注意：在一般情况中，我们对结果进行验证时，需要考虑小数点精确位数的问题。由于文章篇幅限制，这里不做额外的处理。

有了思路以后，我们可以基于 Table Driven Test 实现测试用例：

    func TestSqrt(t *testing.T) {
      var shouldSuccess = []struct {
        input    float64 // input
        expected float64 // expected result
      }{
        {math.Inf(1), math.Inf(1)}, // positive infinity
        {math.Inf(-1), math.NaN()}, // negative infinity
        {-1.0, math.NaN()},
        {0.0, 0.0},
        {-0.0, -0.0},
        {1.0, 1.0},
        {4.0, 2.0},
      }
      for _, ts := range shouldSuccess {
        if actual := Sqrt(t.input); actual != ts.expected {
          t.Fatalf("Sqrt(%f) should be %v, but is:%v\n", ts.input, ts.expected, actual)
        }
      }
    }

辅助函数 (test helper)
----------------------

在写测试的过程中，我们可能遇到下面几个场景：

1.  待测试的功能需要一些前提条件，比如初始化数据库连接、打开文件、创建资源

2.  核心功能测试结束后，需要一些清理工作，比如关闭文件、销毁资源

3.  待测试的功能错误分类比较多，考虑到table driven
    test，写到一个测试函数里可读性比较差

这时候，我们需要定义一些辅助函数，以协助核心功能的测试。下面我们以用户登录校验为例，来看如何使用辅助函数。\
我们要测试的函数是
login，为了保证本次单元测试不会污染数据库，我们采取的流程是：

1.  初始化数据库连接（类似于 Junit 中的 @Before)

2.  创建一个用户 （类似于 Junit 中的 @Before）

3.  测试 login

4.  删除该用户（类似于 Junit 中的 @After)

确定了测试的逻辑以后，我们看下代码：

    // file name: user_test.go
    // source code: https://github.com/oscarzhao/blogger-server/blob/master/controllers/user_test.go

    // package level initialization of database connections
    func init() {
      // init database connections
    }

    // testCreateUser 创建一个临时用户（test helper）
    // 具体流程：
    // 1. mocks a http server
    // 2. send create user request to the server
    func testCreateUser(t *testing, userSpec map[string]string) (int, []byte) {
      // mock a http server
      router := denco.New()
      router.Build([]denco.Record{
        {"/api/v1/users/:user_id", &route{}},
      })

      testURL := "/api/v1/users/" + userID
      _, params, found := router.Lookup(testURL)
      if !found {
        t.Fatalf("fails to look up the route, url:%s\n", testURL)
      }

      handler := func(w http.ResponseWriter, r *http.Request) {
        CreateUser(w, r, params)
      }

      marshaled, _ := json.Marshal(userSpec)
      // create request
      req, err := http.NewRequest("POST", "http://anything.com", bytes.NewBuffer(marshaled))
      if err != nil {
        t.Fatalf("should create user success, but fails to send request, error:%s\n", err)
      }

      // mock ResponseWriter
      w := httptest.NewRecorder()
      // call create operation
      handler(w, req)
      return w.Code, w.Body.Bytes()
    }

    // testDeleteUser 根据 userID 删除一个用户（test helper）
    func testDeleteUser(t *testing.T, userID string) (int, []byte) {
      ...
    }

    // TestVerifyLogin 创建用户、测试登录，然后删除该用户
    // 该函数由 go 语言的 test 框架调用
    func TestVerifyLogin(t *testing.T) {
      userID := uuid.NewV4().String()
      data := map[string]string{
        "username": "simple_yyxxzz",
        "password": "simple_password",
        "email":    "not@changed.com",
        "phone":    "1234567890",
      }
      statusCode, msg := testCreateUser(t, userID, data)
      if statusCode >= http.StatusBadRequest {
        t.Fatalf("should succeeed, create user (%s), but fails, error:%s\n", userID, msg)
      }
      // 测试结束时，清理数据
      defer func(userID string) {
        statusCode, msg := testDeleteUser(t, userID)
        if statusCode >= http.StatusBadRequest {
          t.Errorf("should delete user(%s) successfully, but fails, status code:%d, error:%s\n", userID, statusCode, msg)
        }
      }(userID)

      // 测试登录功能
      shouldSuccess := xxx
      for _, ts := range shouldSuccess {
        statusCode, msg = testVerifyPassword(t, ts)
        if statusCode != http.StatusOK {
          // if use fatal, user will not be cleaned up
          t.Errorf("should verify with %v successfully, but failed, status code:%d, error:%s\n", ts, statusCode, msg)
          return
        }
      }
    }

在测试代码中，我们推荐使用 t.Fatalf ， 而不是
t.Errorf，一方面测试代码不需要做太多容错，另一方面增加了测试代码的可读性。

临时文件
--------

如果待测试的功能模块涉及到文件操作，临时文件是一个不错的解决方案。go语言的
ioutil 包提供了 TempDir 和\
TempFile 方法，供我们使用。

我们以 etcd 创建 wal 文件为例，来看一下 TempDir 的用法：

    // github.com/coreos/etcd/wal/wal_test.go

    func TestNew(t *testing.T) {
      p, err := ioutil.TempDir(os.TempDir(), "waltest")
      if err != nil {
        t.Fatal(err)
      }
      defer os.RemoveAll(p)  // 千万不要忘记删除目录

      w, err := Create(p, []byte("somedata"))
      if err != nil {
        t.Fatalf("err = %v, want nil", err)
      }
      if g := path.Base(w.tail().Name()); g != walName(0, 0) {
        t.Errorf("name = %+v, want %+v", g, walName(0, 0))
      }
      defer w.Close()

      // 将文件 waltest 中的数据读取到变量 gb []byte 中 
      // ...

      // 根据 "somedata" 生成数据，存储在变量 wb byte.Buffer 中
      // ...

      // 临时文件中的数据（gb）与 生成的数据（wb）进行对比
      if !bytes.Equal(gd, wb.Bytes()) {
        t.Errorf("data = %v, want %v", gd, wb.Bytes())
      }
    }

上面这段代码是从 etcd 中摘取出来的，源码查看 [coreos/etcd -
Github](https://github.com/coreos/etcd/blob/2353cbca719f6661c8642d666dd8e16591f5ebb5/wal/wal_test.go)。\
需要注意的是，使用 [TempDir](https://golang.org/pkg/io/ioutil/#TempDir)
和 [TempFile](https://golang.org/pkg/io/ioutil/#TempFile)
创建文件以后，需要自己去删除。

关于 package
------------

在写单元测试时，一般情况下，我们将功能代码和测试代码放到同一个目录下，仅以后缀
\_test 进行区分。

对于复杂的大型项目，功能依赖比较多时，通常在跟目录下再增加一个 test
文件夹，不同的测试\
放到不同的子目录下面，如下图所示：

针对自己的项目进行测试时，可以结合这两种方式实现测试用例，提高代码的可读性和可维护性。

### 相关链接：

1.  [golang.org/pkg/testing](https://golang.org/pkg/testing/)

2.  [Testing Techniques](https://talks.golang.org/2014/testing.slide)

3.  [Table Driven
    Test](https://github.com/golang/go/wiki/TableDrivenTests)

4.  [Learn Testing](https://github.com/golang/go/wiki/LearnTesting)

扫码关注微信公众号“深入Go语言”



