---
title: java并发（10）：高并发扩容
type: tags
tags:
  - null
date: 2019-03-04 17:00:02
categories:
description:
---

# 扩容

占用内存大小取决于工作内存内变量的多少与大小

单个线程占用的内存不会很大，但是很多线程占用内存就会很多

- 垂直扩容：提高系统部件能力（加内存）
- 水平扩容：增加更多系统成员来实现（加服务器）

## 扩容-数据库

读操作扩展

- 垂直扩容
- memcache

- redis
- CDN等缓存

写操作扩展

- 水平扩容
- Cassandra

- Hbase等

# 参考 #

1. 
