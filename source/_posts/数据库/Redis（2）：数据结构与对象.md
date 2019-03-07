---
title: Redis（2）：数据结构与对象
type: tags
tags:
  - null
date: 2019-03-04 18:34:18
categories:
description:
---

# 数据结构与对象

Redis数据库里面的每个键值对都是由对象组成的

- 数据库键总是一个字符串对象
- 数据库键的值可以是
  - 字符串对象
  - 列表对象
  - 哈希对象
  - 集合对象（set）
  - 有序集合对象

# 简单动态字符串SDS

Redis的字符串与传统字符串（以空字符结尾的字符数组，C字符串）不同，Redis自己构建了一种简单动态字符串（SDS）的抽象类型，并用作默认字符串表示。

- 传统字符串在Redis当中只会作为字符串字面量用在一些无须对字符串进行修改的地方。
- 当需要一个可以被修改的字符串值时，就会使用SDS来表示字符串值

SDS使用

- 保存数据库的字符串值
- 用作缓冲区
  - AOF模块中的AOF缓冲区
  - 客户端状态中的输入缓冲区

## SDS定义

```
struct sdshdr{
//记录buf数组已经使用的字节的数量
    int len;
    //记录未被使用字节的数量
    int free;
    //保存字符串
    //最后一个字节保存空字符'\0'，不记在len当中
    char buf[];
}
```

## 与C字符串的区别

C字符串：以长度N+1的字符数组来表示长度为N的字符串，并且字符数组的最后一个元素总是空元素'\0'

C字符串的字符串表示方法不能满足Redis对字符串在安全性、效率以及功能方面的要求

- C字符串需要常数复杂度获取字符串长度。
  - 它不记录自身长度，程序必须遍历整个字符串才能获取长度
- C字符串容易造成缓冲区溢出。
  - C语言字符串拼接函数strcat，假定用户在执行函数时分字符串分配了足够多的内存，如果内存不够，则会缓冲区溢出
    - 这是因为字符串本身不记录其本身的内存信息
  - SDS有free与len，可以知道自身剩余内存大小
- 减少修改字符串时带来的内存重分配次数
  - C字符串每次修改时，都要对保存字符串的数组进行一次内存重分配操作
    - 如果扩展字符串，则需要先分配内存；如果截断，则需要释放不适用的空间，否则会内存泄露。
    - 也因此需要进行系统调用，较为费时
  - SDS空间预分配优化策略
    - 优化字符串增长操作，当需要对SDS进行空间扩展时，程序不仅会为SDS分配必要空间，还会为SDS分配额外的未使用空间
      - 如果SDS修改后，len<1MB，则程序分配和len同样大小的未使用空间
      - 如果len>=1MB，则分配1MB的未使用空间
  - SDS惰性空间释放优化策略
    - 优化缩短操作，程序并不立即使用内存重分配来回收多出来的字节，而是使用free记录下来，等待将来使用
    - 同时提供了相应API，在需要时刻真正释放空间
- 二进制安全
  - C字符串的字符必须包含某种编码，并且不能有空字符。因此只能保存文本，不能保存图片等二进制数据
  - SDS的API都以处理二进制的方式来处理SDS里的数据
- 兼容**部分**C字符串函数

## SDS的主要操作API

![1551695057543](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695057543.png)

![1551695070215](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695070215.png)

# 链表

## 应用

- 列表键的底层实现之一
  - 当一个列表键包含了数量较多的元素，或者列表中包含的元素都是比较常的字符串时，会使用链表作为链表键的底层实现
- 发布与订阅
- 慢查询
- 监视器
- 保存多个客户端的状态信息
- 构建客户端输出缓冲区

## 实现

```C
typedef struct listNode{
	
    struct listNode *prev;
    struct listNode *next;
    //节点的值
    void *value;
}listNode;

typedef struct list{
	
    listNode *head;
    listNode *tail;
    //长度
    unsigned long len;
    //节点值赋值函数 dup
    void *(*dup)(void *ptr);
    //节点释放函数 free
    void *(*free)(void *ptr);
    //节点值对比函数 match
    void *(*match)(void *ptr,void *key);;
}list;
```

特性

- 双端
- 无环
  - 表头的prev与表尾的next都指向null
- 表头指针与表尾指针
- 长度计数器
- 多态
  - void*保存节点值，可以保存各种不同类型的值
  - 使用三个属性可以为节点值设置类型特定函数

## API

![1551695605130](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695605130.png)

![1551695615751](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695615751.png)

# 字典

# 跳跃表

# 整数集合

# 压缩列表

# 对象

# 参考

1. 《Redis设计与实现》

