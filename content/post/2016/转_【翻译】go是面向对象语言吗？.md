
---
date: 2016-12-31T11:34:39+08:00
title: "【翻译】go是面向对象语言吗？"
description: ""
disqus_identifier: 1485833679121523377
slug: "【fan-yi-】goshi-mian-xiang-dui-xiang-yu-yan-ma-？"
source: "https://segmentfault.com/a/1190000001832282"
tags: 
- 翻译 
- golang 
topics:
- 编程语言与开发
---

原文：<http://spf13.com/post/is-go-object-oriented>

前言
----

为了真正理解面向对象的含义，我们需要回顾一下这个概念的起源。第一个面向对象语言-simula问世于19世纪60年代。它引入了对象（object)、类（class）、继承（inheritance）、子类（subclass）、虚方法（virtual
method）、协程（coroutine）等概念。然而simula最重要的贡献可能是它引入颠覆性的思想——将数据和逻辑完全分离。

你可能不熟悉simula语言，但你肯定熟悉Java, C++, C\# &
Smalltalk中的一种，这些语言深受simula的影响，当然这些语言又同时影响着现今几乎所有的高级语言如：Objective
C, Python, Ruby, Javascript, Scala, PHP, Perl…
大部分程序员都遵循着将数据和逻辑完全分离的原则。

由于面向对象没有标准的定义，为了讨论的方便，接下来我们将提供一个标准的定义。

> 面向对象系统将数据和代码通过“对象”集成到一起，而不是将程序看成由分离的数据和代码组成。对象是数据类型的抽象，它有状态（数据）和行为（代码）

面向对象包括继承、多态、虚派生等特性，接下来我们将看看go语言是怎样处理对象、多态、继承，相信读完接下来的介绍，您会对go是如何处理面向对象有自己的见解。

go中的对象
----------

go语言中没有对象(object)这个关键词。对象（object）仅仅是一个单词，重要的是它所表示的含义。尽管go中没有object这种类型，但是go中的struct有着跟object相同的特性。

> struct是一种包含了命名域和方法的类型

让我们从一个例子中来理解它：
```Go
    type rect struct {
        width int
        height int
    }

    func (r *rect) area() int {
        return r.width * r.height
    }

    func main() {
        r := rect{width: 10, height: 5}
        fmt.Println("area: ", r.area())
    }
```
我们一行行来解释一下上面的代码。代码的第一块定义了一个叫做rect的struct类型，该struct含有两个int类型的域；接下来定义了一个绑定在rect
struct类型上的area方法。严格来说，area方法是绑定在指向rectct
struct的指针上。如果方法绑定在rect
type而非指针上，则在调用方法的时候需要使用该类型的值来调用，即使该值是空值，本例的空值实际是一个nil值；代码的最后一块是main函数，main函数第一行创建了一个rect类型的值,当然也有其他的方法来创建一个类型的值，这里给出的是一个地道的方法。main函数的最后一行是打印作用在r值上的area方法的返回结果。

通过上面的描述，可以看出这很像对象的行为，我们可以创建一个结构化的数据类型，然后定义方法和这些数据进行交互。上述的简单例子并没有完成展示面向对象的所有特性，比如继承和多态。需要说明的是go不仅可以在struct上定义方法，在任何命名的类型上同样也可以。比如，可以定义一个名为Counter的新类型，该类型是int型的别名，然后在Counter类型上定义方法。例子详见：<http://play.golang.org/p/LGB-2j707c>

继承和多态
----------

定义对象间的关系的方法有如下几种，它们之间都有一些差别，但目的都是一样的：复用代码。

-   单继承（Inheritance）
-   多继承（Multiple Inheritance）
-   多态（Subtyping/Polymorphism）
-   对象组合（Object composition）

继承：一个对象基于另外一个对象，使用其实现。有两种不同的继承实现：单继承和多继承。它们的不同在于对象是继承自一个对象还是多个对象。单继承关系是一棵树，而多继承关系是一个格状结构。单继承语言包括PHP、C\#、Java、Ruby等，多继承语言包括Perl、Python、C++等

多态
----

多态是is-a的关系，继承是实现的复用。多态定义了两个对象的语义关系，继承定义两个对象的语法关系。

对象组合
--------

对象组合是一个对象包含了其他对象，而非继承，它是has-a的关系，而非is-a。

go语言的继承
------------

go有意得被设计为没有继承语法。但这并不意味go中的对象（struct
value)之间没有关系，只不过go的作者选择了另外一种机制来暗含这种特性。实际上go的这种设计是一种非常好的解决方法，它解决了围绕着继承的数十年的老问题和争论。

最好不要继承
------------

下面引用的一段来自javaworld的一篇名为《[why extends is
evil](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html)》的文章说明了这一点。

> The Gang of Four Design Patterns book discusses at length replacing
> implementation inheritance (extends) with interface inheritance
> (implements).

> I once attended a Java user group meeting where James Gosling (Java’s
> inventor) was the featured speaker. During the memorable Q&A session,
> someone asked him: “If you could do Java over again, what would you
> change?” “I’d leave out classes,” he replied. After the laughter died
> down, he explained that the real problem wasn’t classes per se, but
> rather implementation inheritance (the extends relationship).
> Interface inheritance (the implements relationship) is preferable. You
> should avoid implementation inheritance whenever possible.

go语言中的多态和组合
--------------------

go语言严格遵守[composition over inheritance
principle](http://en.wikipedia.org/wiki/Composition_over_inheritance)的原则。go通过在struct和interface上使用组合和多态来实现继承关系。\
Person和Address之间的关系是这种实现的一个很好的例子：<http://play.golang.org/p/LigPIVT2mf>。
```Go
    type Person struct {
       Name string
       Address Address
    }

    type Address struct {
       Number string
       Street string
       City   string
       State  string
       Zip    string
    }

    func (p *Person) Talk() {
        fmt.Println("Hi, my name is", p.Name)
    }

    func (p *Person) Location() {
        fmt.Println("I’m at", p.Address.Number, p.Address.Street, p.Address.City, p.Address.State, p.Address.Zip)
    }

    func main() {
        p := Person{
            Name: "Steve",
            Address: Address{
                Number: "13",
                Street: "Main",
                City:   "Gotham",
                State:  "NY",
                Zip:    "01313",
            },
        }

        p.Talk()
        p.Location()
    }
```
**Output**

> Hi, my name is Steve\
> I’m at 13 Main Gotham NY 01313

上面的例子需要注意的是，
Address仍然是一个不同的对象，只不过存在于Person中。

go中的伪多态
------------

> AUTHORS NOTE:\
> In the first version of this post it made the incorrect claim that Go
> supports the is-a relationship via Anonymous fields. In reality
> Anonymous fields appear to be an is-a relationship by exposing
> embedded methods and properties as if they existed on the outer
> struct. This falls short of being an is-a relationship for reasons now
> provided below. Go does have support for is-a relationships via
> interfaces, covered below. The current version of this post refers to
> Anonymous fields as a pseudo is-a relationship because it looks and
> behaves in some ways like subtyping, but isn’t.

我们通过扩展上面的例子来说明go中的伪多态。注意这里“伪”字说明实际上go是没有多态的概念的，只不过伪多态表现得像多态一样。下面的例子中，Person可以说话(Talk），一个Citizen也同时是一个Person，因此他也能说话(Talk）。在上面的例子中加入如下内容，完整代码见：<http://play.golang.org/p/eCEpLkQPR3>。
```Go
    type Citizen struct {
       Country string
       Person
    }

    func (c *Citizen) Nationality() {
        fmt.Println(c.Name, "is a citizen of", c.Country)
    }

    func main() {
        c := Citizen{}
        c.Name = "Steve"
        c.Country = "America"
        c.Talk()
        c.Nationality()
    }
```
上面的例子通过引入匿名域（Person）实现了is-a关系。Person是Citizen的一个匿名域（anonymous
field），匿名域只给出了对象类型，而不给出类型的名字。通过匿名域，Citizen可以访问Person中的所有属性（域）和方法。

匿名域方法提升
--------------

上述例子，Citizen可以和Person执行一样的Talk()方法。但如果想要Citizen的Talk()表现出不同的行为该怎么做呢？我们只需要在Citizen上定义方法Talk()即可。当调用c.Talk()的时候，调用的则是Citizen的Talk()方法而非Person的Talk()方法，<http://play.golang.org/p/jafbVPv5H9>。
```Go
    func (c *Citizen) Talk() {
        fmt.Println("Hello, my name is", c.Name, "and I'm from", c.Country)
    }
```
**Output**

> Hello, my name is Steve and I'm from America\
> Steve is a citizen of America

为何匿名域不是合适的多态实现
----------------------------

有两个原因：\
1. 匿名域仍然能被访问，就好像它们是被嵌入的对象一样。\
这并不是一件坏事，多继承存在的一个问题就是当多个父类具有相同的方法的时候，会产生歧义。然而go语言可以通过访问跟匿名类型同名的属性来访问嵌入的匿名对象。实际上当使用匿名域的时候，go会创建一个跟匿名类型同名的对象。上面的例子中，修改main方法如下，我们能很清楚得看出这一点：
```Go
    func main() {
    //    c := Citizen{}
        c.Name = "Steve"
        c.Country = "America"
        c.Talk()         // <- Notice both are accessible
        c.Person.Talk()  // <- Notice both are accessible
        c.Nationality()
    }
```
**Output**

> Hello, my name is Steve and I'm from America\
> Hi, my name is Steve\
> Steve is a citizen of America

1.  真正的多态，派生对象就是父对象\
    如果匿名对象能实现多态，则外层对象应该等同于嵌入的对象，而实际上并非如此，它们仍然是不同的存在。下面的例子印证了这一点：

```Go
    package main

    type A struct{
    }

    type B struct {
        A  //B is-a A
    }

    func save(A) {
        //do something
    }

    func main() {
        b := B
        save(&b);  //OOOPS! b IS NOT A
    }
```
**Output:**

> prog.go:17: cannot use b (type \*B) as type A in function argument\
> \[process exited with non-zero status\]

go中的真正的多态实现
--------------------

> Go interfaces are pretty unique in how they work. This section focuses
> only on how they pertain to subtyping which will not do them proper
> justice. See the further reading section at the end of the post to
> learn more.

正如我们上面提到的，多态是一种is-a的关系。在go语言中，每种类型(type)都是不同的，一种类型不能完全等同于另外一种类型，但它们可以绑定到同一个接口（interface）上。接口能用于函数（方法）的输入输出中，因而可以在类型之间建立起is-a的关系。

go语言定义一个接口并不是使用using关键字，而是通过在对象上定义方法来实现。在[Effective
Go](http://golang.org/doc/effective_go.html#interfaces_and_types)中指出，这种关系就像“如果某个东西能做这件事，那么就把它应用到这里”（不管黑猫白猫，只要能抓到老鼠，我就养这只猫）。这一点很重要，因为这允许一个定义在package外的类型也能实现该接口。

我们接着上面的例子，增加一个新函数SpeakTo，然后修改main函数，将该方法应用到Citizen和Person上,<http://play.golang.org/p/lvEjaMQ25D>。
```Go
    func SpeakTo(p *Person) {
        p.Talk()
    }

    func main() {
        p := Person{Name: "Dave"}
        c := Citizen{Person: Person{Name: "Steve"}, Country: "America"}

        SpeakTo(&p)
        SpeakTo(&c)
    }
```
**Output**

> Running it will result in\
> prog.go:48: cannot use c (type \*Citizen) as type \*Person in function
> argument\
> \[process exited with non-zero status\]

跟预期的结果一样，编译失败。Citizen并不是Person类型，尽管他们拥有同样的属性。然而我们定义一个接口（interface）Human，然后将这个接口作为SpeakTo函数的输入参数，上面的例子就可以正常运行了，<http://play.golang.org/p/ifcP2mAOnf>。
```Go
    type Human interface {
        Talk()
    }

    func SpeakTo(h Human) {
        h.Talk()
    }

    func main() {
        p := Person{Name: "Dave"}
        c := Citizen{Person: Person{Name: "Steve"}, Country: "America"}

        SpeakTo(&p)
        SpeakTo(&c)
    }
```
**Output**

> Hi, my name is Dave\
> Hi, my name is Steve

关于go语言中的多态，有如下两点需要注意。\
1.
可以把匿名域绑定到一个接口，也能绑定到多个接口。接口和匿名域一起使用，可以起到和多态同样的效果。\
2.
go提供了多态的能力。接口的使用能使得实现了该接口的不同对象都能作为函数的输入参数，甚至作为返回结果，但它们仍然保持了它们自己的类型。这点从上面的例子能看出来，我们不能直接在初始化Citizen对象的时候设置Name值，因为Name不是Citizen的属性，而是Person的属性，因而不能再初始化Citizen的时候设置Name值。

go，一个没有object和inheritance的面向对象的语言
-----------------------------------------------

如上所述，面向对象的基本概念在go中被很好的实现了，虽然术语上存在差别。go把struct作为数据和逻辑的结合。通过组合(composition)，has-a关系来最小化代码重用，并且避免了继承的缺陷。go使用接口(interface)来建立类型（type）之间的is-a关系。

欢迎进入无对象的OO编程模型世界！

讨论
----

Join the discussion on [hacker
news](https://news.ycombinator.com/item?id=7868485) and [Reddit -
Golang](http://www.reddit.com/r/golang/comments/27p2bc/is_go_an_object_oriented_language_spf13com/)

深入阅读
--------

<http://nathany.com/good/>\
<http://www.artima.com/lejava/articles/designprinciples.html>\
<http://www.goinggo.net/2014/05/methods-interfaces-and-embedded-types.html>

