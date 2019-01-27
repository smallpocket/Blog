---
title: javaBase异常
type: tags
tags:
  - JavaBase
date: 2018-09-23 10:42:01
categories: java
description: java笔记的基础知识整理，SE异常处理
---
# 异常

面对异常：

1. 向用户报告错误
2. 保存所有的工作结果
3. 允许用户以妥善的形式推出程序

异常分类

![1541174115450](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1541174115450.png)

首先是Error类，它代表硬件上的错误，着运行时系统内部错误或者资源耗尽，或者内存不足等问题，在一些低配般服务器上很可能出现。

更需要关注的是Exception上的错误，它代表着软件方面的错误，也是我们程序员的领域。可以看到一个是RuntimeException类，另一个是IOException。

IO异常，即你在读取U盘，你把U盘拔了，这种搞事情的，或者IO错误等等，一般不可控。

RuntimeException错误，这就是程序错误了

### try-catch



- 对于try-catch-finally
- finally一定会执行,即使遇到了在try或者catch中遇到了return,也会先跳到finally里面执行,然后再跳转回到finally
- 多catch下,catch是顺序执行的,而且由于异常也是一个对象,因此应该将子类的catch放在最顶部,
###throws



- 抛出异常是躲避异常的方法
- 异常会抛给调用该方法的方法,因此,调用方法的方法也要去抛出或者处理异常