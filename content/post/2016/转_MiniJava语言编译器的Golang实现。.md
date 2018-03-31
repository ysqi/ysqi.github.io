
---
date: 2016-12-31T11:34:18+08:00
title: "MiniJava语言编译器的Golang实现。"
description: ""
disqus_identifier: 1485833658216831612
slug: "MiniJavayu-yan-bian-yi-qi-de-Golangshi-xian-。"
source: "https://segmentfault.com/a/1190000003760722"
tags: 
- 编译原理 
- 编译器 
- golang 
categories:
- 编程语言与开发
---

Dog-comp
========

Abstract
--------

文章介绍一个编译器的实现流程。源语言选用MiniJava.
MiniJava是一个面向对象语言，支持继承，对象创建等面向对象的特性。具体语法参考[Tiger
book](http://www.cs.princeton.edu/~appel/modern/java/) 附录。\
[Dog-comp](https://github.com/qc1iu/dog-comp)是一个用golang实现的minijava编译器，目前可以将minijava翻译成c。dog-comp包括前端的lexer，parser，type-checking，codegen，后端还有基于minijava-AST的优化，基于控制流图的优化。为了满足minijava面向对象的特性，Dog-comp还带有一个runtime，实现了一个垃圾收集器。Dog-comp可以作为学习编译器和语言优化的一个小工具。

lexer
-----

虽然目前已经有了词法分析器的生成器，但是Dog-comp使用了转移图(TDA)算法手工实现lexer。

parser
------

Dog-comp的parser使用LL(1)。在Exp与Stm的处理上进行了文法改写，避免了二义性。

elaborator
----------

编译器是一个典型的现行结构，类型检查的输入就是parser的输出，也就是源语言的Ast。

codegen
-------

在代码生成的环节，将源语言的Ast翻译成了C语言的Ast，时候将C-Ast进行打印，实际上就完成了翻译。但是，这里有几个地方需要注意。将一个面向对象的语言翻译成C这样的语言，我们需要在中间做一些变换，这些变换实际上也就是面向对象语言的实现技术。

### 什么是对象？

这里要说对象到底是什么。对象里面可以有数据，还可以有函数，所以我们可以说对象是数据和操作这些数据的函数的集合。换到C语言里面，对象不能仅仅用一个structure表示，这里也就出现了[虚函数表](https://en.wikipedia.org/wiki/Virtual_method_table)的概念。我们可以将数据和一个函数表组合成一个structure，这个structure其实就是java的对象。

### Flattening

所谓的平坦化，就是将java的Class全部消除。显然，在C里面，所有的函数都是平坦的。

垃圾收集器
----------

编译生成的C文件和runtime.c一起编译成可执行文件，使得c带有垃圾收集的功能。为了支持垃圾收集，必须在c的structure里面在加入一些字段，用于GC记录对象的状态。比如需要记录这个对象中那些字段是引用，这个对象是普通对象还是数组对象等等。

优化！
------

[Dog-comp](https://github.com/qc1iu/dog-comp)在源码Ast级别上做了4个优化。

-   死类删除

-   死代码删除

-   代数化简

-   常量折叠

### DeadClass Elimination

\
如上图所示，class
Fac是一个根本没有被用到的类，经过死类删除之后，Fac被干掉了。

### DeadCode Elimination

\
上图的程序包含着死代码。显然，if的else分支是不可能执行的，最后一个while也没有执行的可能。所以经过死代码删除，这两段代码会直接消失。\
\
这里有一个问题需要注意，while(true){}和while(false){}是否应该同样对待呢？while(false)显然是死代码，但是while(true)是否应该被删除呢？答案是否定。

### AlgSimp

代数化简主要是针对运算，比如i=a*b*0\*(a+2)/m),右侧的表达式可以直接优化为0.

### ConstFold

常量折叠是在编译期把常量上的运算做完。\
\
上图的代码中有很多可以优化的地方。执行完常量折叠之后代码如下图。\
\
这里可以观察到，进行完常量折叠之后，实际上有产生的死代码，程序可以继续优化。所以这4个优化不停循环的作，直到达到不动点为止。

### CFG

控制流图是一种方便优化的中间表示。如下图就是[LinkedList.java](https://github.com/qc1iu/dog-comp/blob/master/test/LinkedList.java#L34)中Equal函数的控制流图。\

可以使用

    dog-comp/$ ./dog-comp ../test/LinkedList.java -visualize svg 生成。

