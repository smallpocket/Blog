---
title: 系统log
type: tags
tags:
  - null
date: 2018-09-24 00:30:22
updated: 2018-09-24 00:30:22
categories:
description:
---
# 日志框架 #

## 介绍 ##

- 能实现日志输出的工具包
- 日志：能够描述系统运行状态的所有时间
1. 用户下线
2. 接口超时
3. 数据库崩溃
4. 等等

### 能力 ###

- 定制输出目标
- 定制输出格式
- 携带上下文信息（线程，调用对象）
- 运行时选择性输出
- 灵活的配置
- 优异的性能

### 日志级别 ###

### 框架种类 ###

#### 日志门面 ####

- JCL
- SLF4J 和 logback 是同一个作者，优选

#### 日志实现 ####

- log4j2 极度的性能，太先进，不受到支持
- logback

## 使用 ##
 
	1. private final Logger logger =LoggerFactory.getLogger("类名".class)
	2. @Slf4j
		public class 类名

### 注意 ###

- 使用类名的原因是，报出错误是属于哪个类

## 配置 ##

### 需求 ###

- 区分info和error
- 每天产生一个日志文件
- 
### 配置文件 ###

- application.yml  简单，只能做基础的配置
- logback-spring.xml

#### application ####

	logging:
		path: /a/a/a 配置日志输出路径
		file: /a/a/a 配置日志文件输出路径
		level: debug日志级别
		  com.test.dem: debug指定某个类的debug日志

#### logback-spring ####
18min
	<appender>
		<layout>
			<pattern>

 

# 参考 #
1. 
