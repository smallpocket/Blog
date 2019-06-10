---
title: 集合框架
type: tags
tags:
  - JavaBase
date: 2019-01-26 21:34:14
categories: java
description: java集合框架
---

# 基本概念

![1548671884363](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1548671884363.png)

## 使用原因

编程时候面临一种问题，即在运行时才知道拥有多少对象，需要根据运行时的状态去创建对象，如此，依靠命名创建对象，以获得对对象的引用便不现实。

数组的长度受到限制，不能快速变更，也不是一个很好地选择

java为该问题提供了一套较完整的容器类：List ,Set,Queue,Map

## 分类

1. Collection,一个独立元素序列,包含List,Set,Queue

- 包含了***序列的概念***(这是什么意思鸭),一种存放一组对象的方式

2. Map,一组成对的键值对对象,允许使用键查找值,

## 共性

- 对于Colletcion，遍历的方法可以是foreach，也可以使用迭代器

## Collection接口

在接口当中有一个Iterator的方法，即迭代器，使用迭代器访问集合当中的元素

```java
public interface Collection<E>{
    boolean add(E element);
    Iterator<E> iterator();
    ...
}
public interface Iterator<E>{
    E next();
    boolean hasNext();
    void remove();
    default void forEachRemaining(Consumer<? super E>);
}
```



# 数据结构

## 散列集

- 可以快速查找元素的一种数据结构，但无法控制元素出现的次序
- 通过为每个对象计算一个整数（散列码）来实现，散列码由对象的实例域产生。不同的对象应当产生不同的散列码并与equals方法兼容

java实现

![1549162679421](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1549162679421.png)

java采用链表数组实现，每个列表称为桶。计算对象的位置方法：根据对象的散列码m，与桶的数量n，则m%n便是桶的编号，将对象插入桶中。如果桶满了，则为散列冲突，用对象与桶中对象进行比较，判断是否存在

java SE8，桶满后会从链表转为平衡二叉树

# 集合框架

## List

### 特性

- 以特定的顺序保存一组元素
- 

### ArrayList

针对随机访问元素

### LinkedList

- 针对经常插入或删除中间元素所设计的高效率集合
- 添加了可以使之用作栈、队列、双端队列的方法

linkedList.add方法将对象添加到末尾，而如果想将对象添加到中间，则需要使用迭代器

## Set

### 特性

- 对于每个值,只保存一个对象
- 通常使用set，只关心某事物是否是set的成员

### TreeSet

- 以有序状态保持并防止重复，因为有序，所以对象必须能够相互比较，它们必须实现Comparable接口，或者提供一个Comporable
- 数据存储在红-黑树当中

TreeSet 实现了NavigableSet的接口，增加了几个便于定位元素以及反向遍历的方法

### HashSet

- 防止重复的集合，可以快速寻找到相符合的元素
- 散列存储
- contain方法快速查看是否某个元素已存在，只在某个桶当中查找元素，不必查看集合所有元素

### LinkedHashSet

- 按照被添加的顺序保存对象
- 散列存储

### EnumSet



## Queue

### 特性

- 只允许在容器的一端插入，另一端移除
- Stack，Queue等都是由linkedList支持的

### 分类

- Deque  Queue  ArrayQueue
- 优先级队列，基于堆（可以自我调整的二叉树）实现，元素可以按照任意顺序插入，却总是按照排序的顺序进行检索。即不论何时调用remove方法，总会获得当前优先级队列中最小的元素。（没有进行排序），可以提供comparator对象

## Stack

## Map

### 特性

- 允许将某些对象与其他一些对象关联起来的关联数组，产生对象之间的一种映射
- 可以用成对的name/Value进行保存与取出，想要检索一个对象，必须使用一个键
- 通过Map，可以扩展到多维度，例如Map《Map,Map》,Map 《Person,List》等等

### 操作

1. 更新映射项

存在一个map：count，用于统计一个单词在文件中出现的频度。则使用counts.put(word,counts.get(word)+1);

存在异常，即当第一次发现该单词，counts.get便会返回null的异常

解决方案：

1. merge方法，counts.merge(word,1,Integer::sum);将word与1关联，否则使用Integer::sum函数组合求值
2. counts.getOrDefault(word,0)，

2. counts.putIfAbsent(word,0);couts.put(word,counts.get(word)+1)，该方法只有当键原先存在时才会放入一个值

   

### HashMap

- 只对键进行散列，散列或者比较函数只能作用于键，与键关联的值不能进行散列或者比较

### LinkedHashMap

类似HashMap，但可以记住元素插入的顺序，也可以设定成依照元素上次存取的先后来排序

### TreeMap

- 用键的整体顺序对整体元素进行排序，并将其组织成搜索树

### WeakHashMap

### IdentityHashMap

- 键的散列值不依靠hashCode，而是System.identityHashCode()，是根据内存地址计算散列码，并且比较使用==而不是equals

# 参考 #

1. 《java编程思想 第4版》 第11章 持有对象
2. 《java编程思想 第4版》 第17章 容器深入研究
