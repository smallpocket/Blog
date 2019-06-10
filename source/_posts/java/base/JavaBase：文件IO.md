---
title: java文件IO
type: tags
tags:
- JavaBase
date: 2018-09-23 10:39:13
categories: java
description: java笔记的基础知识整理，SE文件IO
---
# 概述

Java 的 I/O 大概可以分成以下几类：

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

## 磁盘操作

File 类可以用于表示文件和目录的信息，但是它不表示文件的内容。

递归地列出一个目录下所有文件：

```java
public static void listAllFiles(File dir) {
    if (dir == null || !dir.exists()) {
        return;
    }
    if (dir.isFile()) {
        System.out.println(dir.getName());
        return;
    }
    for (File file : dir.listFiles()) {
        listAllFiles(file);
    }
}
```

从 Java7 开始，可以使用 Paths 和 Files 代替 File。

## 字节操作



## 文件输出格式

- 序列化数据
- 纯文本数据
>
- 序列化数据:一般人无法阅读,在纯文本下阅读无意义
- 容易被程序阅读,比较安全
- 纯文本数据可以被其他程序(数据库等)引用
##文件保存
>
- 当保存数据的时候,比如保存一个对象,在对象里对其他对象的引用会使得被引用的对象被序列化,这些对象也会被保存
- 若某个变量不应该被序列化,用标记符transient修饰
- 要记得关闭输入输出流
- 对象具有serialVersionUID,是根据类的结构信息计算得来,对象在解序列化时,会比对对象与要还原成类的UID,如果不符合就抛出异常
- 可以在类里面static final这个ID