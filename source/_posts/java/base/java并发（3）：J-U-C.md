---
title: java并发（三）：J.U.C
type: tags
tags:
  - null
date: 2019-02-28 20:08:11
categories:
description:
---

![1551617354545](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551617354545.png)

AbstractQueuedSynchronizer-AQS



底层数据结构

- 双向链表
- Condition queue，单向链表
  - 当使用到condition时存在，并可能存在多个

设计

- 使用Node实现FIFO队列，可以用于构建锁或者其他同步装置的基础框架
- 利用一个int类型表示状态
  - state变量，表示获取锁的线程锁，>1表示重入锁的数量
- 使用方法是继承
  - 复写其中方法
- 子类通过继承并通过实现它的方法管理其状态（acquire和release）的方法操纵状态
- 可以同时实现排它锁和共享锁模式（独占、共享）

实现思路

- 内部维护一个队列来管理锁
- 线程尝试获取锁
  - 如果失败，将当前线程以及等待状态信息包装为一个node节点加入同步队列
  - 不断尝试获取锁，只有head的直接后继才会尝试，如果失败则阻塞自己
  - 当持有锁的线程释放锁的时候，唤醒head的直接后继

AQS同步组件

- CountDownLatch

- Semaphore
- CyclicBarrier
- ReentrantLock  锁
- Condition
- FutureTask

## CountDownLatch

同步辅助类

应用场景：

- 

## Semaphore

信号量

## CyclicBarrier

## ReentrantLock  与锁

## FutureTask

# 参考 #

1. 
