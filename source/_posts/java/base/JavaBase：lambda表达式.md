---
title: lambda表达式
type: tags
tags:
  - javaBase
date: 2018-10-18 23:16:02
categories: java
description: lambda表达式的使用，以及函数式接口
---
# lambda表达式

lambda表达式是一个可传递的代码块，可以在以后执行一次或多次。

## 适用范围

那么lambda表达式适合处理哪些问题呢

在某些情况中，需要把一段代码传递给某个对象，但是，由于java是面向对象的，所以，这段代码需要包装在一个类当中

举例：
```java
/**
 * 需要一个数组参数与一个实现了Comparator接口的类
 */
public static <T> void sort(T[] a, Comparator<? super T> c) 
```

因此，如果我们想要去使用这个sort，那么就需要去构造一个实现了该接口的类，然后去重写方法，将这个类传递给sort才行。

``` java
/**
 * 构造一个实现Comparator接口的类
 */
class LengthComparator implements Comparator<String>{
  public int compareTo(String f,String s){
    return f.length()-s.length();
  }
}
```

如此,我们才能够实现按照字符串长度来排序。

可以看到的是，在这个类当中内容并不是很多，只是一个代码块而已。sort()需要的也只是那个compareTo方法。

## 使用lambda

那么最终我们需要的代码块是：
```java 
f.length()-s.length();
//java作为强类型,则需要指明类型:
(String f,String s)
  ->f.length()-s.length()
//以上便是一个完整的lambda表达式
//完整的调用一次
Arrays.sort(planets, (String first,String second) 
    -> first.length() - second.length());
```

lambda既然是代码块，当然也就不只一行

```java 
(String f,String s)->{
  //随便写,就像一个方法一样
  return f.length()-s.length();
}
```

### 一些特殊写法

经常会看到一些和上面的示例并不相同的写法

1. 没有写类型参数

``` java
/**
 * planets是一个String数组
 * 因此编译器可以推导出其后面的应当是一个String类型的参数
 * 所以省略了参数
 */
Arrays.sort(planets, (first,second) 
    -> first.length() - second.length());
```

2. 没有写()

```java
/**
 * 方法只有一个参数,并且参数类型可以推导出来
 */
ActionListener listener = event - >
System.out.println ( " The time is " + new Date ( ) " ) ;
```

## 处理lambda表达式

lambda表达式的重点是“延时执行”，之所以如此是因为
- 在一个单独的线程运行代码
- 多次运行代码
- 在算法的适当位置运行代码（例如排序）
- 发生某种情况时运行代码
- 只在必要时候运行

### 注意

即便这个lambda不需要任何的方法参数，依然要写()

```java 
()->{
  int i=0;
  System.out.println(i);
}
```

lambda表达式可以访问外围方法或类当中的变量，但是这些变量必须是最终变量，不能随意改变，也不可以在表达式内改变

# 函数式接口

## 定义

对于只有一个抽象方法的接口，需要这种接口的对象时就可以提供一个lambda表达式，这种接口为函数式接口。

## 方法引用

对于已经存在的方法，可以完成想传递给其他代码的某个动作，代替了代码块，则可以使用方法引用

```java 
Timer t = new Timer ( 1000 , System.out:: println );

//等价于:
Timer t = new Timer ( 1000 ,x-> System.out.println(x) );
```

使用:
- object :: instanceMethod
- Class :: staticMethod
- Class :: instanceMethod

# 参考 #
1. 
