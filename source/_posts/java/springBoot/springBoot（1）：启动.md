---
title: springBootStart
date: 2018-09-22 19:45:59
type: "tags" 
categories: "springBoot"
tags:        
    - springBoot
    - start
description: 初学springBoot，帮助搭建spring boot以及基础特性
---
# 基础搭建 #
## 运行环境 ##
	- JDK 7 或 8及以上
	- Maven 3.0+
	- 编译器：IDEA

## 搭建 ##
可参考课程

### 配置 ###

#### 自动化配置 ####

- spring boot相比较于Spring是做了一些约定，对于Spring所需要的配置，它进行了默认的配置，可以适应大多数的业务场景

#### 多环境配置 ####

- 文件application-{}.properties对应不同的环境的配置文件。
- 使用语句调用不同的配置文件
    spring.profiles.active=dev 

#### 路径配置

为所有的URL添加前缀：

        server: 
            servlet:
                context-path: /api

#### 自定义配置 ####

Spring的配置有一个配置的优先级
1. 命令行参数
2. java:comp/env 里的 JNDI 属性
3. JVM 系统属性
4. 操作系统环境变量
5. RandomValuePropertySource 属性类生成random.* 属性
6. 应用以外的 application.properties（或 yml）文件
7. 打包在应用内的 application.properties（或 yml）文件
8. 在应用 @Configuration 配置类中，用@PropertySource 注解声明的属性文件
9. SpringApplication.setDefaultProperties 声明的默认属性

踩坑
- 关于中文，properties在配置中文值会出现乱码，因为编码是 iso-8859，可以改用yml进行配置

#### 关于yml ####

其配置方法与properties不太一样，特此说明

      home:
      	province: 浙江省
      	city: 温岭松门
     	 desc: 我家住在${home.province}的${home.city}

---

## 运行 ##

### 启动类 ###
    /**
     * Spring Boot应用启动类
     *
     * Created by bysocket on 16/4/26.
     */
    @SpringBootApplication
    public class Application {
 
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
    }
1. @SpringBootApplication：Spring Boot 应用的标识
2. Application很简单，一个main函数作为主入口。SpringApplication引导应用，并将Application本身作为参数传递给run方法。具体run方法会启动嵌入式的Tomcat并初始化Spring环境及其各Spring组件。

### 跑起来 ###

- 在IDEA当中Spring的运行是有一个专门的SpringBoot运行方法
- 运行端口默认为8080，修改端口的路径为src/main/resources/application.properties,修改server.port=8888即可

# 参考 #
1. [2小时学会Spring Boot](https://www.imooc.com/learn/767 "2小时学会Spring Boot")
  
