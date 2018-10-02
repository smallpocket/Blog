---
title: Aop
date: 2018-09-23 00:01:03
tags:
---
## 使用AOP处理请求 ##
### AOP介绍 ###
- AOP是一种编程范式
- 与语言无关，是一种程序设计思想
- AOP中文为面向切面编程
- 面向对象关注的是将功能垂直地，并且相互独立的，并封装为良好的类，让他们拥有自己的行为
- AOP利用一种“行切”的技术 ，将面向对象构成的庞大的类的体系进行水平的切割，并且将那些影响到了多个类的公共行为封装起来，形成一个可重用的模块，这个模块称为切面
- 关键思想：将通用的逻辑从业务的逻辑中分离出来

![业务生命周期](https://i.imgur.com/16lbw1i.png)
![AOP](https://i.imgur.com/IGyDR61.png)

### AOP实战 ###
进行权限的控制：需要登录，之后才能进行访问
- 添加依赖 spring-boot-starter-aop
- 新建包aspect
- 新建类，为类添加注解@Aspect
- 新建方法，添加注解@Before，在方法执行之前进行执行。Before参数为（“execution()”）,execution参数为（包名。类名。方法）
- 方法可以是*，则会拦截全部的方法，方法的参数可以是..，如此对于方法任意参数都会被拦截


- 对方法的注解：
- @Before在之前执行，@After在之后执行
- @PointCut标志一个切面
- @AfterReturning，获取方法的返回值
> returning：入参 

#### @PointCut ####

- 定义一个公共的方法，拦截一个切面，@PointCut("execution()")，方法为log()
- 对于Before去针对这个切面，只需要Before("log()")

