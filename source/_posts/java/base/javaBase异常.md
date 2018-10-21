---
title: javaBase异常
type: tags
tags:
  - JavaBase
date: 2018-09-23 10:42:01
categories: java
description: java笔记的基础知识整理，SE异常处理
---
###try-catch
>
- 对于try-catch-finally
- finally一定会执行,即使遇到了在try或者catch中遇到了return,也会先跳到finally里面执行,然后再跳转回到finally
- 多catch下,catch是顺序执行的,而且由于异常也是一个对象,因此应该将子类的catch放在最顶部,
###throws
>
- 抛出异常是躲避异常的方法
- 异常会抛给调用该方法的方法,因此,调用方法的方法也要去抛出或者处理异常