---
title: javaBase线程
type: tags
tags:
- JavaBase
date: 2018-09-23 10:39:50
categories: java
description: java线程初学习
---
###建立线程
- runnable(这是个接口)对象:线程的任务,该任务在线程的执行空间运行
- thread对象:执行工人,赋值任务
- 启动thread
###runnable
- runnable只有一个方法run,也就是只能执行一个任务
###线程锁synchronized
- 线程可以在任何时候sleep,任何时候醒来
- 线程锁有一定的成本,查询钥匙具有一定性能上的损耗
- 线程锁会导致因为同步化的方法速度减慢   
死锁
- 当两个线程互相持有对方所希望的钥匙,则会出现死锁,相互僵持
###线程
- 线程具有一定的成本