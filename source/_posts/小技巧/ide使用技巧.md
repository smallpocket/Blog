---
title: ide使用技巧
type: tags
tags:
  - ide
date: 2018-09-28 09:36:34
updated: 2018-09-28 09:36:34
categories: 插件
description: IDEA的一些使用技巧、插件等
---
# idea

## 快捷键

### 自动代码格式化

>Ctrl+alt+L

### 插入getter和setter

>alt+insert

## 插件篇

### 直接修改源码，不需要重新启动 ##

>jetbrain插件

### 不用书写get/set方法 ##

>lombok插件

1. 添加依赖

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

2. 添加插件lombok
3. 添加类注解@Data 或 @Get

### 编码规范

> save action插件

> Alibaba Java Coding Guidelines插件

# vscode技巧

- 在当前目录打开命令行：Ctrl+shift+c

# markdown使用

## 如何添加代码块

使用格式:

\``` 某语言 (java,c,c++)   
代码块  
\```


### 添加超链接

 \[浏览器上显示的内容](超链接 "鼠标悬停显示的文字")

# 参考 #
1.  [Intellij Idea 代码格式化/保存时自动格式化](https://blog.csdn.net/mr_rain/article/details/79279931)
2. [idea 离线安装 lombok插件](https://blog.csdn.net/shmily_lsl/article/details/80689307 "idea 离线安装 lombok插件")
3. [vscode 在当前文件位置打开控制](https://blog.csdn.net/curious_again/article/details/78560023)
