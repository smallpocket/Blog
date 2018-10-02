---
title: bootDao
date: 2018-09-23 00:00:44
tags:
---
# 数据库操作 #

- 使用Mysql和Spring-Data-Jpa
- Spring-Data-Jpa是对Hibernate的整合
- JPA定义了一系列对象持久化的标准
- 实现了这一标准的产品有Hibernate等
- 
## 数据库配置 ##

	datasource:
		driver-class-name: com.mysql.jdbc.Driver
    	url: jdbc:mysql://127.0.0.1:3306/boot
    	username: root
    	password: 123456
	jpa:
    	hibernate:
      	ddl-auto: create
    show-sql: true

### ddl-auto ###

- create：每次删除表，然后创建一个新表
- update：如果存在数据，则会保留数据 
- validate:验证表里面的属性是否与结构一致，不一致则报错

## 数据库操作 ##

- 基于JPA进行操作
- JpaRepository<Gril,Integer>
- Gril为类型，Integer为主键ID的类型

### 查 ###

1. 基于主键
查询全部记录
findAll方法
	grilRepository.findAll()
查询单个记录
findOne(id)方法

2. 基于其他
在JPA接口当中定义方法 findBy'某字段'，例如
	public List<Gril> findByAge(Integer age)
调用方法 findByAge(age)

### 增 ###

增加一个记录
save方法，返回值为添加的对象
	grilRepository.save()

### 删 ###

删除一个记录
delete方法，返回空

### 改 ###

更新一个记录
save方法

## 事务 ##

@Transactional
