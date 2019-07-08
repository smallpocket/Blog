---
title: Linux：概述
type: tags
tags:
  - null
date: 2019-06-27 16:55:47
categories:
description:
---

# Linux发展史

## 分类

- 内核版本：内核官网（www.kernel.org）

- 外核版本
  - 在内核基础上做出改进，即发行版本。

### 发行版本

- redhat：它有一部分收费的售后功能
- Centos：是完全免费的，并且与redhat一致
- fediro：个人版
- Ubuntu：图形界面更优秀
- debian：

Linux应用领域

# 开源软件

开源软件，即获得程序的源代码，而不是编译后的二进制文件

- 使用的自由：绝大多数开源软件免费
- 研究的自由：可以获得软件源代码
- 散步及改良的自由：可以自由传播、改良甚至销售

在Linux当中，以服务器端的角度看，Linux当中有更多的开源软件，有着更高质量的软件

- Apache：网站服务搭建软件
- Nginx：占用服务器资源更少，支持更高的并发
- MySQL
- php
- Samba
- mongoDB
- python
- Ruby
- Sphinx等

# Linux应用领域

### 基于Linux的企业服务器

www.netcraft.com

### 嵌入式应用

# XShell

ctrl l清屏

# Linux常用命令

tab键会对Linux命令进行补全。

两下tab会以该命令开头的所有命令打印出

## 命令基本格式

命令提示符

[root@localhost ~]#

- root：当前登录用户
- localhost：主机名
- ~：当前所在目录
  - 用户登录的家目录。root用户在/root
  - 普通用户在/home/user1/
- #（$）：超级（普通）用户的提示符

### 命令格式

命令 [选项] [参数]

注意：

- 个别命令使用不遵循此格式。
- 当有多个选项时，可以写在一起
- 简化选项与完整选项
  - -a在一些目录当中等于--all

## 基本命令

### pwd命令

显示当前所在目录：print working directory

### ls

查询目录中的内容。ls [选项] [文件或目录]

选项

- -a 显示所有文件，包括隐藏文件

- -l 显示详细属性

  - 第一列代表权限。默认为10位 -rw-r--r--。

    - 第一位：文件类型（- 文件 d 目录 I 软链接文件，即快捷方式）
    - 2-4件：文件的所有者，u。r：读，w：写，x：执行。
    - 5-7位：文件的所属组，g。相同用户相同权限的人放到一组。
    - 8-10位：其他人，o，不能使用。

  - ```shell
    -rwxr-xr-x  1 root root      347472 Jun 29  2017 xfs_copy
    ```

  - 1 代表引用该文件的次数

  - 347472是字节

  - Jun 29  2017代表最后修改的时间

- -d 查看目录属性

- -h 人性化显示文件大小

- -i 显示inode

## 文件处理命令

***目录处理命令***

### mkdir

建立目录：mkdir -p [目录名]

- -p 递归创建
- 英文名 make directory

### cd

切换目录：cd [目录]change directory

- cd ~ 回到家目录
- cd - 进入上次所在目录
- cd .. 进入上级目录
- cd . 进入当前目录

相对路径进行查找 cd ../usr/local

绝对路径：要从根目录进行制定，cd /etc/

### rmdir

只能删除空白目录：remove empty directory

### rm 

删除文件或目录：rm [选项] [文件或目录]：remove

- -r：删除目录
- -f：强制执行 false

### cp

cp [选项] [原文件或目录] [目标目录]：复制命令copy

- -r 复制目录
- -p 连带文件属性复制
- -d 若源文件是链接文件，则复制链接属性
- -a 相当于-pdr

### mv

mv [源文件或目录] [目标目录]。剪切或改名命令：move

***文件处理命令***



***链接命令***



## 文件搜索命令

## 帮助命令

## 压缩命令

## 关机命令

## 其他常用命令

# 参考 #

1. 
