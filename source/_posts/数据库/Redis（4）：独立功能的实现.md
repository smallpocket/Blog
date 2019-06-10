---
title: Redis（4）：独立功能的实现
type: tags
tags:
  - NoSQL
  - Redis
date: 2019-03-04 18:37:23
categories: 数据库
description:
---

# 事务

Redis通过MULTI（开始）、EXEC（提交）、WATCH等实现事务功能

事务提供将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制，在事务执行期间，服务器不会中断事务执行其他客户端的命令请求。

事务的原子性、一致性、隔离性、持久性 ACID

- **Atomicity（原子性）**：一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被[回滚](https://zh.wikipedia.org/wiki/%E5%9B%9E%E6%BB%9A_(%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86))（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。
- **Consistency（一致性）**：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设[约束](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%AE%8C%E6%95%B4%E6%80%A7)、[触发器](https://zh.wikipedia.org/wiki/%E8%A7%A6%E5%8F%91%E5%99%A8_(%E6%95%B0%E6%8D%AE%E5%BA%93))、[级联回滚](https://zh.wikipedia.org/w/index.php?title=%E7%BA%A7%E8%81%94%E5%9B%9E%E6%BB%9A&action=edit&redlink=1)等。
- **Isolation（隔离性）**：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **Durability（持久性）**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## 事务的实现

事务的三个阶段

- 事务开始
  - MULTI命令
- 命令入队
- 事务执行

## WATCH

乐观锁，在EXEC命令执行前，监视任意数量的数据库键，并在EXEC命令执行时，检查被监视的键是否至少有一个已经被修改，如果是，则拒绝执行事务，并返回代表事务执行失败的空回复

# 参考 #

1. 
