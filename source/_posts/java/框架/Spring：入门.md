---
title: Spring：入门
type: tags
tags:
  - 框架
  - Spring
date: 2019-03-26 20:59:40
categories: Java
description:
---

# Spring介绍

Spring诞生：

- 创建Spring的目的就是用来**替代更加重量级的的企业级Java技术**
- 简化Java的开发
  - 基于POJO轻量级和**最小侵入式开发**
  - 通过依赖注入和面向接口实现**松耦合**
  - **基于切面**和惯例进行声明式编程
  - 通过切面和模板**减少样板式代码 **

***侵入式概念***

**侵入式框架**

- 对于EJB、Struts2等一些传统的框架，

  通常是要实现特定的接口，继承特定的类才能增强功能

  - **改变了java类的结构**

**非侵入式**

- 对于Hibernate、Spring等框架，**对现有的类结构没有影响，就能够增强JavaBean的功能**

***松耦合***

前面我们在写程序的时候，都是**面向接口编程，通过DaoFactroy等方法来实现松耦合**

![这里写图片描述](assets/1621e4da347ed640)

DAO层和Service层**通过DaoFactory来实现松耦合**

- 如果Serivce层直接new DaoBook()，那么DAO和Service就紧耦合了【Service层依赖紧紧依赖于Dao】

而Spring给我们更加合适的方法来实现松耦合，并且更加灵活、功能更加强大！---->**IOC控制反转**

***切面编程***

切面编程也就是AOP编程，其实我们在之前也接触过...**动态代理就是一种切面编程了**...

当时我们使用**动态代理+注解的方式给Service层的方法添加权限**.

```Java
    @Override
    @permission("添加分类")
    /*添加分类*/
    public void addCategory(Category category) {
        categoryDao.addCategory(category);
    }


    /*查找分类*/
    @Override
    public void findCategory(String id) {
        categoryDao.findCategory(id);
    }

    @Override
    @permission("查找分类")
    /*查看分类*/
    public List<Category> getAllCategory() {
        return categoryDao.getAllCategory();
    }

    /*添加图书*/
    @Override
    public void addBook(Book book) {
        bookDao.addBook(book);

    }
```

- Controller调用Service的时候，Service返回的是一个代理对象
- 代理对象得到Controller想要调用的方法，通过反射来看看该方法上有没有注解
- 如果有注解的话，那么就判断该用户是否有权限来调用 此方法，如果没有权限，就抛出异常给Controller，Controller接收到异常，就可以提示用户没有权限了。

AOP编程可以简单理解成：**在执行某些代码前，执行另外的代码**

- Struts2的拦截器也是面向切面编程【在执行Action业务方法之前执行拦截器】

Spring也为我们**提供更好地方式来实现面向切面编程**！

## Spring概述

Spring 是个java企业级应用的开源开发框架。Spring主要用来开发Java应用，但是有些扩展是针对构建J2EE平台的web应用。Spring 框架目标是简化Java企业级应用开发，并通过POJO为基础的编程模型促进良好的编程习惯。

### spring优点

- **轻量：**Spring 是轻量的，基本的版本大约2MB。
- **控制反转：**Spring通过控制反转实现了松散耦合，对象们给出它们的依赖，而不是创建或查找依赖的对象们。
- **面向切面的编程(AOP)：**Spring支持面向切面的编程，并且把应用业务逻辑和系统服务分开。
- **容器：**Spring 包含并管理应用中对象的生命周期和配置。
- **MVC框架**：Spring的WEB框架是个精心设计的框架，是Web框架的一个很好的替代品。
- **事务管理：**Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全局事务（JTA）。
- **异常处理：**Spring 提供方便的API把具体技术相关的异常（比如由JDBC，Hibernate or JDO抛出的）转化为一致的unchecked 异常。

### Spring模块组成

- 核心容器
  - Core module
  - Bean module
  - Context module
  - Expression Language module
- 数据集成/访问。提供与数据库交互的支持
  - JDBC module
  - ORM module
  - OXM module
  - Java Messaging Service(JMS) module
  - Transaction module
- Web应用程序的支持
  - Web module
  - Web-Servlet module
  - Web-Struts module
  - Web-Portlet module
- AOP
- Instrumentation：类检测和类加载器支持
- Test
- 杂项
  - Messaging
  - Aspects

### 什么是 Spring 配置文件？

Spring 配置文件是 XML 文件。该文件主要包含类信息。它描述了这些类是如何配置以及相互引入的。但是，XML 配置文件冗长且更加干净。如果没有正确规划和编写，那么在大项目中管理变得非常困难。

### Spring 应用程序有哪些不同组件？

Spring 应用一般有以下组件：

- **接口** - 定义功能。
- **Bean 类** - 它包含属性，setter 和 getter 方法，函数等。
- **Spring 面向切面编程（AOP）** - 提供面向切面编程的功能。
- **Bean 配置文件** - 包含类的信息以及如何配置它们。
- **用户程序** - 它使用接口。

### 使用 Spring 有哪些方式？

使用 Spring 有以下方式：

- 作为一个成熟的 Spring Web 应用程序。
- 作为第三方 Web 框架，使用 Spring Frameworks 中间层。
- 用于远程使用。
- 作为企业级 Java Bean，它可以包装现有的 POJO（Plain Old Java Objects）。

# 引出Spring

我们试着回顾一下没学Spring的时候，是怎么开发Web项目的

- 1. **实体类**--->class User{ }
- 2. **daoclass**-->  UserDao{  .. 访问**db}
- 3. **service**--->class  UserService{  UserDao userDao = new UserDao();}
- 4. **actionclass**  UserAction{UserService userService = new UserService();}

**用户访问：**

- **Tomcat->action->service->dao**

我们来思考几个问题：

- ①：**对象创建创建能否写死？**

- ②：对象创建细节 

  - 对象数量

    - ```
              action  多个   【维护成员变量】
      ```

    - ```
              service 一个   【不需要维护公共变量】
      ```

    - ```
              dao     一个   【不需要维护公共变量】
      ```

  - 创建时间

    - ```
              action    访问时候创建
      ```

    - ```
              service   启动时候创建
      ```

    - ```
              dao       启动时候创建
      ```

- ③：对象的依赖关系 

  - **action 依赖 service**
  - **service依赖 dao**

对于第一个问题和第三个问题，**我们可以通过DaoFactory解决掉(虽然不是比较好的解决方法)**

对于第二个问题，我们要**控制对象的数量和创建事件就有点麻烦了**....

而**Spring框架通过IOC就很好地可以解决上面的问题**....

## IOC控制反转

Spring的核心思想之一：**Inversion of Control , 控制反转 IOC**

那么控制反转是什么意思呢？？？**对象的创建交给外部容器完成，这个就做控制反转。**

- Spring使用控制反转来实现对象不用在程序中写死
- 控制反转解决对象处理问题【把对象交给别人创建】

那么对象的对象之间的依赖关系Spring是怎么做的呢？？**依赖注入，dependency injection.即DI**

- Spring使用依赖注入来实现对象之间的依赖关系
- 在创建完对象之后，对象的关系处理就是依赖注入

上面已经说了，控制反转是通过外部容器完成的，**而Spring又为我们提供了这么一个容器，我们一般将这个容器叫做：IOC容器.**

无论是创建对象、处理对象之间的依赖关系、对象创建的时间还是对象的数量，我们都是在Spring为我们提供的IOC容器上配置对象的信息就好了。

那么使用**IOC控制反转这一思想有什么作用呢**？？？我们来看看一些优秀的回答...

来自知乎：[www.zhihu.com/question/23…](https://link.juejin.im?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F23277575%2Fanswer%2F24259844)

我摘取一下核心的部分：

> ioc的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。**第一，资源集中管理，实现资源的可配置和易管理**。**第二，降低了使用资源双方的依赖程度，也就是我们说的耦合度**。
>
> 也就是说，甲方要达成某种目的不需要直接依赖乙方，它只需要达到的目的告诉第三方机构就可以了，比如甲方需要一双袜子，而乙方它卖一双袜子，它要把袜子卖出去，并不需要自己去直接找到一个卖家来完成袜子的卖出。它也只需要找第三方，告诉别人我要卖一双袜子。这下好了，甲乙双方进行交易活动，都不需要自己直接去找卖家，相当于程序内部开放接口，卖家由第三方作为参数传入。甲乙互相不依赖，而且只有在进行交易活动的时候，甲才和乙产生联系。反之亦然。这样做什么好处么呢，甲乙可以在对方不真实存在的情况下独立存在，而且保证不交易时候无联系，想交易的时候可以很容易的产生联系。甲乙交易活动不需要双方见面，避免了双方的互不信任造成交易失败的问题。**因为交易由第三方来负责联系，而且甲乙都认为第三方可靠。那么交易就能很可靠很灵活的产生和进行了**。这就是ioc的核心思想。生活中这种例子比比皆是，支付宝在整个淘宝体系里就是庞大的ioc容器，交易双方之外的第三方，提供可靠性可依赖可灵活变更交易方的资源管理中心。另外人事代理也是，雇佣机构和个人之外的第三方。 ==========================update===========================
>
> 在以上的描述中，诞生了两个专业词汇，依赖注入和控制反转所谓的依赖注入，则是，甲方开放接口，在它需要的时候，能够将乙方传递进来(注入)所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。

参考优秀的博文①：[www.tianmaying.com/tutorial/sp…](https://link.juejin.im?target=https%3A%2F%2Fwww.tianmaying.com%2Ftutorial%2Fspring-ioc)

参考优秀的博文②：[这里写链接内容](https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzAxOTc0NzExNg%3D%3D%26mid%3D2665513179%26idx%3D1%26sn%3D772226a5be436a0d08197c335ddb52b8%23rd)

**知乎@Intopass的回答：**

1. 不用自己组装，拿来就用。
2. 享受单例的好处，效率高，不浪费空间。
3. 便于单元测试，方便切换mock组件。
4. 便于进行AOP操作，对于使用者是透明的。
5. 统一配置，便于修改。

------

## Spring模块

**Spring可以分为6大模块：**

- Spring Core  spring的核心功能： IOC容器, 解决对象创建及依赖关系

- Spring Web  Spring对web模块的支持。 

  - ```
    可以与struts整合,让struts的action创建交给spring
    ```

  - ```
    spring mvc模式
    ```

- Spring DAO  Spring 对jdbc操作的支持  【JdbcTemplate模板工具类】

- Spring ORM  spring对orm的支持： 

  - 既可以与hibernate整合，【session】
  - 也可以使用spring的对hibernate操作的封装

- Spring AOP  切面编程

- SpringEE   spring 对javaEE其他模块的支持



![这里写图片描述](assets/1621e4da3492d633)



上面文主要引出了为啥我们需要使用Spring框架，以及大致了解了Spring是分为六大模块的....**下面主要讲解Spring的core模块！**

# 参考 #

1. 
