---
title: MQL5能运用在A股中吗
slug: mql5-can-use-in-shanghai-a
categories:
  - 金融
tags:
  - MQL5
date: 2015-09-14T08:18:14
disqus_identifier: 100025
---

很久前，就买了个VPN，利用Google搜索获取些难得的信息，而这次却意外收获了一个免费使用的自动化交易利器MQL5,能否利用到国内？能否用于A股交易？
但最终方向，国外的好东东，总不能顺利的移植到国内，记录下一天的收获！


### MQL5是什么
MQL5的全称是： MetaQuotes Software Language 5。 他的意思是： MetaQuotes Software 软件的第五代编程语言，也就是MT5软件的编程语言。 MT5是MetaTrade 5的英文缩写。 它是由MetaQuotes（迈达克）公司编写的一款外汇、期货等金融产品的交易软件/.

具体参考百度百科：[MQL5百度百科](http://baike.baidu.com/link?url=ocFLj1VZYTA-y-Er5yywzmiXQ-celvxUNUexm3ZO2L15yTawCSHpg9Dz_CbmMgvdWVKXPnd5SbiUOxnO7n-pwq)

### MQL5能否用于A股

看官方的文档，其实很早就开始支持股票交易，但是因为国内政治因素，无法进入国内市场，从而无法让委托自动发送到交易所。
而MQL5的机制是，提供客户端给你，你可以自己DIY各种策略，实现定制化的交易信号，再将交易发送到MetaQuots公司，再转到各个交易所。
理论上无法运用到A股


### 如何让MQL5支持国内A股交易
这个命题，不是伪命题，但是目前无法解答，还得需要国内市场的开放性和成熟性来支持。

### 绕个弯子，来实现MQL5支持A股交易

MQL5客户端是支持自定义加载脚本的，通过Editor自行开发策略，提供交易信号后，也通过脚本和国内交易系统对接，
将交易发送到交易系统中，在技术是完成没问题的，只要是程序便可以相互通信。但存在另一个问题：如何在MQL5中接入国内行情？
策略必然会依赖行情，那么我们下一步便是解决MQL5中的行情接入问题！


记录下将MQL5应用到A股的方案，路途曲折，不尽人意，但总有一丝希望！



