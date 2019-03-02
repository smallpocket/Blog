---
title: java并发（一）--基础
type: tags
tags:
  - null
date: 2019-02-26 19:59:31
categories:
description:
---

# 目录

## 基础知识

- 并发、高并发相关概念 
- CPU多级缓存 
- java内存模型
- 并发优势与风险
- 并发模拟

## 并发及并发的线程安全处理：

- 多线程并发编程
  - 线程安全性
    - 原子性、可见性、有序性
    - atomic包
    - CAS算法
    - synchronize与Lock、volatile、happes-before
  - 安全发布对象
    - 安全发布方法
    - 不可变对象、final关键字的使用、不可变方法
    - 线程不安全类与写法
  - 线程封闭、同步容器、并发容器
    - 堆栈封闭
    - ThreadLocal线程封闭、JDBC的线程封闭
    - 同步容器、并发容器
    - J.U.C
  - AQS等J.U.C组件
    - 
  - 线程调度
    - 线程池好处
    - new Thread弊端
    - ThreadPoolExecutor、Executor框架接口
  - 线程安全补充内容
    - 死锁产生于预防
    - 多线程并发的最佳实践
    - spring的线程安全
    - hashmap与concurrentHashMap

## 高并发解决思路与手段

- 扩容
  - 水平扩容
  - 垂直扩容
- 缓存
  - redis
  - memcache
  - guavaCache
- 队列
  - kafka
  - rabbitMQ
  - rocketMQ
- 拆分
  - 服务化Dubbo
  - 微服务spring cloud
- 服务降级与熔断
  - hystrix介绍与使用
  - 服务降级的多种选择
- 数据库切库分库分表
  - 切库、分表、支持多数据源的原理及实现
- 高可用的一些手段
  - 任务调度分布式elastic-job
  - 主备curator的实现
  - 监控报警机制
- 限流
  - guava rateLimiter
  - 常用限流算法
  - 分布式限流

## 知识技能

总体架构：springboot、maven、jdk8、mysql

基础组件：mybatis、guava、lombok、redis 、kafka

高级组件（类）：joda-Time、atomic包、J.U.C、AQS、ThreadLocal、RateLimiter、Hystrix、threadPool、shardbatis、curator、elastic-job

J.U.C java并发编程核心



# 并发、高并发相关概念

并发：同时拥有两个或多个线程，如果程序在单核处理器上运行，多个线程将交替地还如或者换出内存，线程同时存在， 每个线程都处于执行过程中的某个状态，如果运行在多核处理器上，程序的每个线程都将分配到一个处理器核上，因此可以同时运行。

高并发：通过设计保证系统能够同时并行处理很多请求

# CPU多级缓存 

高速缓存的配置：数据的读取与存储都经过高速缓存，CPU核心与高速缓存有一条快速通道。主存与高速缓存都连接在系统总线上，并且该主线还用于与其他通信

![1551250626973](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551250626973.png)

多级缓存：一级缓存的速度与主存速度拉大。加入二级缓存、三级缓存，速度慢于1级，但是更大

![1551250713552](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551250713552.png)

**多级缓存的意义**：

- CPU的频率太快，主存无法跟上，在处理器时钟周期内，CPU常常需要等待主存，浪费资源。缓存是为了缓解CPU与内存间速度不匹配的问题
- 缓存无法存放CPU需要的全部数据，但是由于：
- 时间局部性：如果某个数据被访问，那么在不久的将来可能再次被访问
- 空间局部性：如果某个数据被访问，那么与它相邻的数据很快也可能被访问

**缓存一致性(MESI协议)**

*挠头*

用于保证多个CPU 缓存之间共享数据的一致，定义了cache的四种状态

- M：被修改，该缓存只缓存在CPU的缓存中，并且被修改过，与主存数据不一致，需要在某个时间写回主存（这个时间其他CPU不能读取该数据），被写会主存当中后转换为E。
- E：独享，只缓存在CPU缓存中，但是没有被修改，与主存数据一致，当其他CPU读取该数据，转换为S，当修改该数据则转换为M
- S：共享，该缓存可能被多个CPU共享，并与主存一致，如果有一个CPU修改该缓存，则转换为M。其他CPU的缓存转换为I
- I：无效，

操作：

- local read：读本地缓存数据
- local write：将数据写入本地缓存
- remote read：将主存数据读取过来
- remote write：将数据写入主存

![1551252298293](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551252298293.png)

**乱序执行优化**

处理器为提高运算速度而做出违背代码原有顺序的优化。在不影响结果的情况下，代码的执行次序可能会发生变化

在多核CPU中，后写入缓存的数据未必真的写入缓存。

# java内存模型 JMM

为了屏蔽各种硬件与操作系统的内存访问差异，以实现java程序在各种系统下都能并发。

JMM规范了java虚拟机与计算机内存如何协同工作，规定了一个线程如何和何时能看到其他线程修改过的共享变量的值，以及在必须时如何同步地访问共享变量

**内存分配**

堆：运行时的数据区，由垃圾回收负责，动态分配内存。由于运行时动态分配，存取速度慢

栈：存取速度快，数据可共享。存在栈当中的数据大小与生存期必须确定。

当线程中对对象的引用，引用了在堆上的对象，调用了对象的方法，访问了对象的数据，这个时候他们拥有的是对象成员变量的私有**拷贝**

![1551253396237](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551253396237.png)

**CPU寄存器**

CPU registers，访问缓存的速度最快，其次是高速缓存

![1551253669203](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551253669203.png)

![1551253796357](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551253796357.png)

java内存模型——

## 同步八种操作

- lock锁定：作用于主内存的变量，把一个变量标识为一条线程独占状态
- unlock解锁：作用于主内存的变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
- read读取：作用于主内存的变量，把一个变量值从主内存传输到线程的工作内存中，以便随后load动作使用
- load载入：作用于工作内存的变量，把read操作从主内存得到的变量值放入工作内存的**变量副本**中
- use使用：作用于工作内存的变量，把工作内存的一个变量值传递给**执行引擎**（线程）
- assign赋值：作用于工作内存的变量，把一个从执行引擎接收到的值赋值给工作内存的变量
- store存储：作用于工作内存的变量，把工作内存的一个变量的值传递到主内存中，以便随后的write操作
- write写入：作用于主内存的变量，把store操作从工作内存中一个变量的值传送到主内存的变量中

![1551254279492](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551254279492.png)

## 同步规则

- 不允许read、load与store、write操作单一出现，但是不必连续执行，中间可以插入其他指令
- 不允许一个线程丢弃掉它最近的assign操作，必须将变化同步给主内存
- 不允许一个线程无原因地（没有assign操作）将数据同步给主内存。
- 一个新的变量只能从主内存中诞生，不允许在工作内存直接使用一个未初始化（load或assign）的变量，即在use与store前，必须先assign
- 一个变量在同一时刻只允许一个线程进行lock，lock可以被同一线程执行多次。
- 如果一个变量执行了lock操作，将清空工作内存中次变量的值，在执行引擎使用这个变量前需要重新load或assign操作初始化变量
- 如果一个变量没有lock，则不允许unlock操作，也不能unlock其他线程lock的变量
- 对变量unlock前，必须先将变量同步到主内存

# 并发优势与风险

优势：

- 速度：同时处理多个请求，响应快，复杂操作可以分为多个进程同时执行
- 设计：在某些情况更简单
- 资源利用：CPU在等待IO时可以做其他的事情

风险：

- 安全性：多个线程共享数据可能会产生与预期不相符的结果
- 活跃性：某个操作无法继续执行下去，如死锁、饥饿等
- 性能：线程过多使得CPU频繁切换，调度时间增多；同步机制；消耗过多内存

# 并发模拟

- postman：http请求模拟
- Apache bench：测试网站性能
- JMeter：压力测试工具
- 代码：semaphore信号量、CountDownLatch计数器。通常与线程池共同使用

# 参考 #

1. java并发编程与高并发解决方案
