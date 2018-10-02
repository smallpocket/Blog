---
title: javaBaseInterface
type: tags
date: 2018-09-23 10:30:29
categories: java
tags:
- javaBase
description: java笔记的基础知识整理，SE接口
---
# 接口与抽象类 #

## 抽象类 ##

1.某些类根本无法初始化,比如说animal类,那类似于这种应该作为一个抽象类abstract,或者说是一个接口   
2.抽象类可以拥有抽象方法和非抽象方法    
3.抽象方法没有实体
***
### 实现了接口就履行了合约 ###

1.履行合约的方式是去定义相关的方法  
2.履行了合约,即可去实现与合约相关的动作,实现类似继承和多态

### 关于引用的一些问题 ###

	ArrayList<Object> myDogArrayLust = new ArrayList<Object>();   
	Dog aDog =new Dog(); 
	MyDogArrayList.add(aDog); 
	Dog d=myArrayList.get(0);!!!!报错     

#### 解析 ####

1.不管放进去了什么,出来也只是Object,除非使用了泛型,标明了类型   
2.传递过程,传递进去一个Dog类型,然后由于参数是Object,父类引用子类,成功add,但是取出的时候, return的参数也是Object    
3.因此说,里面确实存放的是一个包含Dog属性的类型,但是获取的却是一个Object的引用,那么根据多态,子类是无法引用父类的    
4.同样,如果调用一个.eat()方法,那么首先这是一个Object类型的引用,编译器会去寻找Object类及其父类(当然是没有了)的eat()方法,很明显的问题,是不存在的  
5.因为对于编译器来说,当你传递进去的时候,它是它原来的类型,取出来的时候,它确实是一个Object类型,而不是引用了其他类型,编译器只根据引用的类型,而不是对象的类型   
***

- 强制类型转换使用instanceof可以检验是否可以成功转换类型`o instanceof Dog`转换成功返回true



