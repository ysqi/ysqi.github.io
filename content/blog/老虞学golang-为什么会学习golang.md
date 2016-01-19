---
date: 2013-04-01
title: 老虞学GoLang-为什么会学习goLang
slug: ysqi-why_studay_golang
topics:
- 开发
tags:
- Go
- 笔记
- 教程
books:
- 老虞学GoLang
disqus_identifier: 100010
---

2009年开始接触软件开发，一直深爱着它，喜爱淘腾些新技术新技能，却至今没有所成。也许专心才能做好一些事，2013年初接触Go Lang,感受着这门语言带来的魅力，自己该在这条路上留下足迹，以此见证自己的成长历程。


### 为什么会学习Go Lang

1.  编程本身是一门艺术，Go Lang 有着无尽想象的魅力。

2. Gmail, Google Search, Google Translate,YouTube 这些已成为我生活工作不可缺少的一部分，足够证明Google的产品是优秀的，同样Go Lang也是优秀的，事实证明确实如此。

3. golang是开源项目，它的社区时活跃的，它的创造者是行业Big牛。

4. golang被创造的目的是明确的：提高开发人员的编程效率，构建服务器软件......


### 一段摘录

http://wiki.ubuntu.org.cn/Golang

**简介**

Go语言是由Google开发的一个开源项目，目的之一为了提高开发人员的编程效率。 Go语言语法灵活、简洁、清晰、高效。它对的并发特性可以方便地用于多核处理器 和网络开发，同时灵活新颖的类型系统可以方便地编写模块化的系统。go可以快速编译， 同时具有垃圾内存自动回收功能，并且还支持运行时反射。Go是一个高效、静态类型， 但是又具有解释语言的动态类型特征的系统级语法。

**应用**

　  由于Go尚未成熟，因此谷歌旗下各类面向用户的服务或应用都没有采用该语言。正因如此，谷歌才需要外部编程人员的协助。
通过创建新的编程语言，谷歌将继续拓展计算领域的各个方面，从而促进这些领域的发展。这同样也是谷歌开发Android操作系统、Chrome浏览器和Chrome OS的动机所在。
北京时间2010年1月10日，Go语言摘得了TIOBE公布的2009年年度大奖。该奖项授予在2009年市场份额增长最多的编程语言。

**功能**

 Google对Go寄予厚望。其设计是让软件充分发挥多核心处理器同步多工的优点，并可解决若干物件取向程序设计的麻烦。它具有现代的程序语言特色，如垃圾回收，帮助程序设计师处理琐碎但重要的内存管理问题。Go的速度也非常快，几乎和C或C++程序一样快，且能够快速制作程序。 　　

Go的网站就是用Go所建立，但Google有更大的野心。该软件是专为构建服务器软件所设计（如Google的Gmail）。Google认为Go还可应用到其他领域，包括在浏览器内执行软件，取代目前JavaScript的角色。 　　

{% blockquote Pike %}
它至少在强度上比JavaScript高一级。Google自建Chrome浏览器，部分原因就是加速JavaScript和网页表现，而Google已经融合了本身的技术，如Native Client和Gears.
{% endblockquote %}

Pike表示，Go另一项与网络相关的特色，是服务器和用户端设备，如PC或手机，可以分担工作。因此，使用Go的服务便可轻松适应不同的用户端处理性能。Go也可解决目前的一大挑战：多核心处理器。一般电脑程序通常依序执行，一次进行一项工作，但多核心处理器更适合同步处理许多工作。Pike说：我们自认有足够的支持，可改善这方面的问题。 　　Go团队正在寻求帮助。其中一个重要领域是改善Go能够使用的runtime library。这类library可提供许多工具和功能，加快程序设计的过程。而Go的library还包括许多重要的设计元素，并供应处理同作、垃圾收集和其他低层杂务的资源。 　　Go团队也需要编译器方面的协助。Thompson曾为32位元和64位元x86处理器，及ARM处理器写过一些编译器，Taylor也为GCC编译器写过一个Go前端。 　　尽管Google对Go有很大的野心，该公司也明白，这项计划无法完全取代现有的技术。Pike说：我不认为我们能取代任何东西。我们只是创造出这个领域的另一个角色。

**特点**

　简洁 快速 安全 并行 有趣 开源 支持泛型编程，内存管理，数组安全，编译迅速


**go语言的开发团队**

　 Thompson：1983年图灵奖（Turing Award）和1998年美国国家技术奖（National Medal of Technology）得主。他与Dennis Ritchie是Unix的原创者。Thompson也发明了后来衍生出C语言的B程序语言。

　　Pike：曾是贝尔实验室（Bell Labs）的Unix团队，和Plan 9操作系统计划的成员。他与Thompson共事多年，并共创出广泛使用的UTF-8 字元编码。

　　Robert Griesemer：曾协助制作Java的HotSpot编译器，和Chrome浏览器的JavaScript引擎V8。

　　此外还有Plan 9开发者Russ Cox、和曾改善目前广泛使用之开原码编译器GCC的Ian Taylor。




学习了一段时间的Go Lang 把我的整理的资源翻出来

官网：www.golang.org ,目前被墙了，

go language Resources :http://go-lang.cat-v.org

一直在更新的文章列表，一日一篇:http://www.miek.nl/files/go ,是否可以去翻译下呢

go百度贴吧：http://tieba.baidu.com/f?kw=golang  ,是否专一呢


godoc翻译和本地化：https://code.google.com/p/go-zh/ ，项目地址：http://go-tour-zh.appsp0t.com

邮件列表: 重要基地

https://groups.google.com/group/golang-china/

https://groups.google.com/group/golang-nuts/

一段介绍Go Lang的视频:http://v.youku.com/v_show/id_XMTY4Mzk5NTc2.html

书籍: 《Go Programming》 《Go 语言编程》

请记住国内的大牛:@许式伟,@ASTA谢



我们已初步认识Go Lang ，那我们将背包上路了。
