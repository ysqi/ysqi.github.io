
---
date: 2017-02-18T11:13:44+08:00
title: "设计模式(golang)"
description: ""
disqus_identifier: 1487387624149976523
slug: "she-ji-mo-shi-(golang)"
source: "https://my.oschina.net/u/1167564/blog/838997"
tags: 
- golang 
topics:
- 编程语言与开发
---

设计模式的六大原则
==================

摘自
[Java开发中的23种设计模式详解](http://www.cnblogs.com/maowang1991/archive/2013/04/15/3023236.html)

-   1、开闭原则（Open Close Principle）

    开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。
    所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后
    面的具体设计中我们会提到这点。

-   2、里氏代换原则（Liskov Substitution Principle）

    里氏代换原则(Liskov Substitution Principle
    LSP)面向对象设计的基本原则之一。 里氏代换原则中说，任何
    基类可以出现的地方，子类一定可以出现。
    LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受
    到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。
    实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽
    象化的具体步骤的规范。—— From Baidu 百科

-   3、依赖倒转原则（Dependence Inversion Principle）

    这个是开闭原则的基础，具体内容：真对接口编程，依赖于抽象而不依赖于具体。

-   4、接口隔离原则（Interface Segregation Principle）

    这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，
    其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

-   5、迪米特法则（最少知道原则）（Demeter Principle）

    为什么叫最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。

-   6、合成复用原则（Composite Reuse Principle）

    原则是尽量使用合成/聚合的方式，而不是使用继承。

实现代码 {#h1_1}
========

[https://github.com/BPing/golang\_design\_pattern/tree/master/pattern](https://github.com/BPing/golang_design_pattern/tree/master/pattern)

-   创建型 C
-   结构型 J
-   行为型 X

``` 
   -AbstractFactory.go : 抽象工厂模式(C)
   -Adapter.go : 适配器模式(J)
   -Bridge.go : 桥接模式(J)
   -Builder.go : 建造者模式(C)
   -ChainOfResponsibility.go : 责任链模式(X)
   -Command.go : 命令模式(X)
   -Composite.go : 组合模式(J)
   -Decorator.go : 装饰器模式(J)
   -Facade.go : 外观模式(J)
   -Factory.go : 工厂模式(C)
   -Flyweight.go : 享元模式(J)
   -Interpreter.go : 解释器模式(X)
   -Iterator.go : 迭代器模式(X)
   -Mediator.go : 中介者模式(X)
   -Memento.go : 备忘录模式(X)
   -Observer.go : 观察者模式(X)
   -Prototype.go : 原型模式(C)
   -Proxy.go : 代理模式(J)
   -Singleton.go : 单例模式(C)
   -Singleton2.go : 单例模式(C)
   -State.go : 状态模式(X)
   -Strategy.go : 策略模式(X)
   -Template.go : 模板模式(X)
   -Visitor.go : 访问者模式(X)
```

