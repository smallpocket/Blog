---
title: Spring：AOP
type: tags
tags:
  - Spring
  - 框架
date: 2019-03-26 21:00:39
categories: Java
description:
---

# AOP产生的背景

为了能够更好地将系统级别的代码抽离出来，去掉与对象的耦合，就产生了面向AOP（面向切面）。如上图所示，OOP属于一种横向扩展，AOP是一种纵向扩展。AOP依托于OOP，进一步将系统级别的代码抽象出来，进行纵向排列，实现低耦合。

## 应用场景

- 日志、安全性、事务等

## AOP 的家庭成员

### PointCut

即在哪个地方进行切入,它可以指定某一个点，也可以指定多个点。

比如类A的methord函数，当然一般的AOP与语言（AOL）会采用多用方式来定义PointCut,比如说利用正则表达式，可以同时指定多个类的多个函数。

### Advice

在切入点干什么，指定在PointCut地方做什么事情（增强），打日志、执行缓存、处理异常等等。

### Advisor/Aspect

PointCut + Advice 形成了切面Aspect，这个概念本身即代表切面的所有元素。但到这一地步并不是完整的，因为还不知道如何将切面植入到代码中，解决此问题的技术就是PROXY

### Proxy

Proxy 即代理，其不能算做AOP的家庭成员，更相当于一个管理部门，它管理 了AOP的如何融入OOP。之所以将其放在这里，是因为Aspect虽然是面向切面核心思想的重要组成部分，但其思想的践行者却是Proxy,也是实现AOP的难点与核心据在。

# AOP的技术实现

AOP仅仅是一种思想，那为了让这种思想发光，必然脱离语言本身的技术支持，Java在实现该技术时就是采用的代理Proxy,那我们就去了解一下，如何通过代理实现面向切面。

- **由于静态代理需要实现目标对象的相同接口，那么可能会导致代理类会非常非常多....不好维护**---->因此出现了动态代理
- 动态代理也有个约束：**目标对象一定是要有接口的，没有接口就不能实现动态代理**.....----->因此出现了cglib代理
- cglib代理也叫子类代理，**从内存中构建出一个子类来扩展目标对象的功能！**
  - **CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。**

## 静态代理

就像我们去买二手房要经过中介一样，房主将房源委托给中介，中介将房源推荐给买方。中间的任何手续的承办都由中介来处理，不需要我们和房主直接打交道。无论对买方还是卖房都都省了很多事情，但同时也要付出代价，对于买房当然是中介费，对于代码的话就是性能。下面我们来介绍实现AOP的三种代理方式。

下面我就以买房的过程中需要打日志为例介绍三种代理方式

静态和动态是由代理产生的时间段来决定的。静态代理产生于代码编译阶段，即一旦代码运行就不可变了。下面我们来看一个例子

```java
public interface IPerson {
    public void doSomething();
}
```

```java
public class Person implements IPerson {
    public void doSomething(){
        System.out.println("I want wo sell this house");
    }
}
```
```java
public class PersonProxy {
    private IPerson iPerson;
    private final static Logger logger = LoggerFactory.getLogger(PersonProxy.class);


public PersonProxy(IPerson iPerson) {
    this.iPerson = iPerson;
}
public void doSomething() {
    logger.info("Before Proxy");
    iPerson.doSomething();
    logger.info("After Proxy");
}

public static void main(String[] args) {
    PersonProxy personProxy = new PersonProxy(new Person());
    personProxy.doSomething();
}

}
```

通过代理类我们实现了将日志代码集成到了目标类，但从上面我们可以看出它具有很大的局限性：需要固定的类编写接口（或许还可以接受，毕竟有提倡面向接口编程），需要实现接口的每一个函数（不可接受），同样会造成代码的大量重复，将会使代码更加混乱。

## 动态代理

那能否通过实现一次代码即可将logger织入到所有函数中呢，答案当然是可以的，此时就要用到java中的反射机制

```java
public class PersonProxy implements InvocationHandler{
    private Object delegate;
    private final Logger logger = LoggerFactory.getLogger(this.getClass();

    public Object bind(Object delegate) {
        this.delegate = delegate;
        return Proxy.newProxyInstance(delegate.getClass().getClassLoader(), delegate.getClass().getInterfaces(), this);
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        try {
            logger.info("Before Proxy");
            result = method.invoke(delegate, args);
            logger.info("After Proxy");
        } catch (Exception e) {
            throw e;
        }
        return result;
    }

    public static void main(String[] args) {
        PersonProxy personProxy = new PersonProxy();
        IPerson iperson = (IPerson) personProxy.bind(new Person());
        iperson.doSomething();
    }
}
```

它的好处理时可以为我们生成任何一个接口的代理类，并将需要增强的方法织入到任意目标函数。但它仍然具有一个局限性，就是只有实现了接口的类，才能为其实现代理。

## CGLIB

CGLIB解决了动态代理的难题，它通过生成目标类子类的方式来实现来实现代理，而不是接口，规避了接口的局限性。
CGLIB是一个强大的高性能代码生成包，其在运行时期（非编译时期）生成被 代理对象的子类，并重写了被代理对象的所有方法，从而作为代理对象。

```java
public class PersonProxy implements MethodInterceptor {
    private Object delegate;
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    public Object intercept(Object proxy, Method method, Object[] args,  MethodProxy methodProxy) throws Throwable {
        logger.info("Before Proxy");
        Object result = methodProxy.invokeSuper(method, args);
        logger.info("After Proxy");
        return result;
    }

    public static Person getProxyInstance() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Person.class);

        enhancer.setCallback(new PersonProxy());
        return (Person) enhancer.create();
    }
}复制代码
```

当然CGLIB也具有局限性，对于无法生成子类的类（final类），肯定是没有办法生成代理子类的。

以上就是三种代理的实现方式，但千成别被迷惑了，在Spring AOP中这些东西已经被封装了，不需要我们自己实现。要不然得累死，但了解AOP的实现原理（即基于代理）还是很有必要的。

# Spring AOP全面认知

## AOP概述

AOP称为面向切面编程，那我们怎么理解面向切面编程？？

我们可以先看看下面这段代码：

![img](assets/1639259ea5e927d8)

我们学Java面向对象的时候，如果代码重复了怎么办啊？？可以分成下面几个步骤：

- 1：抽取成方法
- 2：抽取类

抽取成类的方式我们称之为：**纵向抽取**

- 通过继承的方式实现纵向抽取

但是，我们现在的办法不行：即使抽取成类还是会出现重复的代码，因为这些逻辑(开始、结束、提交事务)**依附在我们业务类的方法逻辑中**！



![img](assets/1639259ea399586b)



现在纵向抽取的方式不行了，AOP的理念：就是将**分散在各个业务逻辑代码中相同的代码通过横向切割的方式**抽取到一个独立的模块中！



![img](assets/1639259ea3e1fbcf)



上面的图也很清晰了，将重复性的逻辑代码横切出来其实很容易(我们简单可认为就是封装成一个类就好了)，但我们要将这些**被我们横切出来的逻辑代码融合到业务逻辑中**，来完成和之前(没抽取前)一样的功能！这就是AOP首要解决的问题了！

## Spring AOP原理

> 被我们横切出来的逻辑代码融合到业务逻辑中，来完成和之前(没抽取前)一样的功能

没有学Spring AOP之前，我们就可以使用代理来完成。

- 代理能干嘛？代理可以帮我们**增强对象的行为**！
- 使用动态代理实质上就是**调用时拦截对象方法，对方法进行改造、增强**！

其实Spring AOP的底层原理就是**动态代理**！

来源《精通Spring4.x 企业应用开发实战》一段话：

> Spring AOP使用纯Java实现，它不需要专门的编译过程，也不需要特殊的类装载器，它在**运行期通过代理方式向目标类织入增强代码**。在Spring中可以无缝地将Spring AOP、IoC和AspectJ整合在一起。

来源《Spring 实战 (第4版)》一句话：

> Spring AOP构建在动态代理基础之上，因此，**Spring对AOP的支持局限于方法拦截**。

在Java中动态代理有**两种**方式：

- JDK动态代理
- CGLib动态代理



![img](assets/1639259ea4fdea1d)



JDK动态代理是需要实现某个接口了，而我们类未必全部会有接口，于是CGLib代理就有了~~

- CGLib代理其生成的动态代理对象是目标类的子类
- Spring AOP**默认是使用JDK动态代理**，如果代理的类**没有接口则会使用CGLib代理**。

那么JDK代理和CGLib代理我们该用哪个呢？？在《精通Spring4.x 企业应用开发实战》给出了建议：

- 如果是**单例的我们最好使用CGLib代理**，如果是多例的我们最好使用JDK代理

原因：

- JDK在创建代理对象时的性能要高于CGLib代理，而生成代理对象的运行性能却比CGLib的低。
- 如果是单例的代理，推荐使用CGLib

看到这里我们就应该知道什么是Spring AOP(面向切面编程)了：**将相同逻辑的重复代码横向抽取出来，使用动态代理技术将这些重复代码织入到目标对象方法中，实现和原来一样的功能**。

- 这样一来，我们就在**写业务时只关心业务代码**，而不用关心与业务无关的代码

## AOP的实现者

AOP除了有Spring AOP实现外，还有著名的AOP实现者：AspectJ，也有可能大家没听说过的实现者：JBoss AOP~~

我们下面来说说AspectJ扩展一下知识面：

> AspectJ是**语言级别**的AOP实现，扩展了Java语言，定义了AOP语法，能够在**编译期**提供横切代码的织入，所以它有**专门的编译器**用来生成遵守Java字节码规范的Class文件。

而Spring借鉴了AspectJ很多非常有用的做法，**融合了AspectJ实现AOP的功能**。但Spring AOP本质上**底层还是动态代理**，所以Spring AOP是不需要有专门的编辑器的~

## AOP的术语

嗯，AOP搞了好几个术语出来~~两本书都有讲解这些术语，我会尽量让大家看得明白的：

**连接点**(Join point)：

- **能够被拦截的地方**：Spring AOP是基于动态代理的，所以是方法拦截的。每个成员方法都可以称之为连接点~

**切点**(Poincut)：

- **具体定位的连接点**：上面也说了，每个方法都可以称之为连接点，我们**具体定位到某一个方法就成为切点**。

**增强/通知**(Advice)：

- 表示添加到切点的一段逻辑代码，并定位连接点的方位信息。 
  - 简单来说就定义了是干什么的，具体是在哪干
  - Spring AOP提供了5种Advice类型给我们：前置、后置、返回、异常、环绕给我们使用！

**织入**(Weaving)：

- 将`增强/通知`添加到目标类的具体连接点上的过程。

**引入/引介**(Introduction)：

- `引入/引介`允许我们**向现有的类添加新方法或属性**。是一种**特殊**的增强！

**切面**(Aspect)：

- 切面由切点和`增强/通知`组成，它既包括了横切逻辑的定义、也包括了连接点的定义。

在《Spring 实战 (第4版)》给出的总结是这样子的：

> 通知/增强包含了需要用于多个应用对象的横切行为；连接点是程序执行过程中能够应用通知的所有点；切点定义了通知/增强被应用的具体位置。其中关键的是切点定义了哪些连接点会得到通知/增强。

总的来说：

- 这些术语可能翻译过来不太好理解，但对我们正常使用AOP的话**影响并没有那么大**~~看多了就知道它是什么意思了。

## Spring对AOP的支持

Spring提供了3种类型的AOP支持：

- 基于代理的经典SpringAOP 

  - 需要实现接口，手动创建代理

- 纯POJO切面 

  - 使用XML配置，aop命名空间

- @AspectJ

  注解驱动的切面 

  - 使用注解的方式，这是最简洁和最方便的！

# 基于代理的经典SpringAOP

这部分配置比较麻烦，用起来也很麻烦，这里我就主要整理一下书上的内容，大家看看了解一下吧，我们实际上使用Spring AOP基本不用这种方式了！

首先，我们来看一下增强接口的继承关系图：



![img](assets/1639259edfbea6f0)

可以分成**五类**增强的方式：

![img](assets/1639259ea8ffc354)

Spring提供了**六种的切点类型**：

![img](assets/163925a0307e7c6e)

**切面类型主要分成了三种**：

- **一般切面**
- **切点切面**
- **引介/引入切面**

![img](assets/1639259f25f81c96)

一般切面，切点切面，引介/引入切面介绍：

![img](assets/1639259f27c9deaf)

![img](assets/1639259f334512f0)

对于切点切面我们一般都是直接用就好了，我们来看看引介/引入切面是怎么一回事：

- 引介/引入切面是引介/引入增强的封装器，通过引介/引入切面，**可以更容易地为现有对象添加任何接口的实现**！

继承关系图：

![img](assets/1639259f3be5cbb2)

引介/引入切面有两个实现类：

- DefaultIntroductionAdvisor：常用的实现类
- DeclareParentsAdvisor：用于实现AspectJ语言的DeclareParent注解表示的引介/引入切面

实际上，我们使用AOP往往是**Spring内部使用BeanPostProcessor帮我们创建代理**。

这些代理的创建器可以分成三类：

- 基于Bean配置名规则的自动代理创建器：BeanNameAutoProxyCreator
- 基于Advisor匹配机制的自动代理创建器：它会对容器所有的Advisor进行扫描，实现类为DefaultAdvisorAutoProxyCreator
- 基于Bean中的AspectJ注解标签的自动代理创建器：AnnotationAwareAspectJAutoProxyCreator

对应的类继承图：

![img](assets/1639259f48e7ef5a)

嗯，基于代理的经典SpringAOP就讲到这里吧，其实我是不太愿意去写这个的，因为已经几乎不用了，在《Spring 实战 第4版》也没有这部分的知识点了。

- 但是通过这部分的知识点可以**更加全面地认识Spring AOP的各种接口**吧~

# 拥抱基于注解和命名空的AOP编程

Spring在新版本中对AOP功能进行了增强，体现在这么几个方面：

- 在XML配置文件中为AOP提供了aop命名空间
- 增加了AspectJ切点表达式语言的支持
- 可以无缝地集成AspectJ

那我们使用`@AspectJ`来玩AOP的话，学什么？？其实也就是上面的内容，学如何设置切点、创建切面、增强的内容是什么...

![img](assets/1639259f3916db2c)

具体的切点表达式使用还是前往：[Spring【AOP模块】就这么简单](https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI4Njg5MDA5NA%3D%3D%26mid%3D2247483954%26idx%3D1%26sn%3Db34e385ed716edf6f58998ec329f9867%26chksm%3Debd74333dca0ca257a77c02ab458300ef982adff3cf37eb6d8d2f985f11df5cc07ef17f659d4%23rd)看吧~~

对应的增强注解：

![img](assets/1639259f856b96ea)

![img](assets/1639259f85f3f536)

## 使用引介/引入功能实现为Bean引入新方法

其实前置啊、后置啊这些很容易就理解了，整篇文章看下来就只有这个引介/引入切面有点搞头。于是我们就来玩玩吧~

我们来看一下具体的用法吧，现在我有个服务员的接口：

```Java
public interface Waiter {

    // 向客人打招呼
    void greetTo(String clientName);

    // 服务
    void serveTo(String clientName);
}
```

一位年轻服务员实现类：

```Java
public class NaiveWaiter implements Waiter {
    public void greetTo(String clientName) {
        System.out.println("NaiveWaiter:greet to " + clientName + "...");
    }

    @NeedTest
    public void serveTo(String clientName) {
        System.out.println("NaiveWaiter:serving " + clientName + "...");
    }

}
```

现在我想做的就是：**想这个服务员可以充当售货员的角色，可以卖东西**！当然了，我肯定不会加一个卖东西的方法到Waiter接口上啦，因为这个是暂时的~

所以，我搞了一个售货员接口：

```Java
public interface Seller {

  // 卖东西
  int sell(String goods, String clientName);
}
```

一个售货员实现类：

```Java
public class SmartSeller implements Seller {

	// 卖东西
	public int sell(String goods,String clientName) {
		System.out.println("SmartSeller: sell "+goods +" to "+clientName+"...");
		return 100;
	}
	
}
```

此时，我们的类图是这样子的：

![img](assets/1639259f88bcecc8)

现在我想干的就是：**借助AOP的引入/引介切面，来让我们的服务员也可以卖东西**！

我们的引入/引介切面具体是这样干的：

```Java
@Aspect
public class EnableSellerAspect {
    
    @DeclareParents(value = "com.smart.NaiveWaiter",  // 指定服务员具体的实现
            defaultImpl = SmartSeller.class) // 售货员具体的实现
    public Seller seller; // 要实现的目标接口
    
}
```

写了这个切面类会发生什么？？

- 切面技术将SmartSeller融合到NaiveWaiter中，这样**NaiveWaiter就实现了Seller接口**！！！！

是不是很神奇？？我也觉得很神奇啊，我们来测试一下：

我们的`bean.xml`文件很简单：

```Java
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">
    <aop:aspectj-autoproxy/>
	<bean id="waiter" class="com.smart.NaiveWaiter"/>
	<bean class="com.smart.aspectj.basic.EnableSellerAspect"/>
</beans>
```

测试一下：

```Java
public class Test {
    public static void main(String[] args) {


        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("com/smart/aspectj/basic/beans.xml");
        Waiter waiter = (Waiter) ctx.getBean("waiter");

        // 调用服务员原有的方法
        waiter.greetTo("Java3y");
        waiter.serveTo("Java3y");

        // 通过引介/引入切面已经将waiter服务员实现了Seller接口，所以可以强制转换
        Seller seller = (Seller) waiter;
        seller.sell("水军", "Java3y");

    }
}
```

![img](assets/1639259fa6f87c3a)

具体的调用过程是这样子的：

> 当引入接口方法被调用时，代理对象会把此调用委托给实现了新接口的某个其他对象。实际上，一个Bean的实现被拆分到多个类中



![img](assets/1639259faa1e4069)

# 总结

看起来AOP有很多很多的知识点，其实我们只要记住AOP的核心概念就行啦。

下面是我的简要总结AOP：

- AOP的底层实际上是动态代理，动态代理分成了JDK动态代理和CGLib动态代理。如果被代理对象没有接口，那么就使用的是CGLIB代理(也可以直接配置使用CBLib代理)
- 如果是单例的话，那我们最好使用CGLib代理，因为CGLib代理对象运行速度要比JDK的代理对象要快
- AOP既然是基于动态代理的，那么它只能对方法进行拦截，它的层面上是方法级别的
- 无论经典的方式、注解方式还是XML配置方式使用Spring AOP的原理都是一样的，只不过形式变了而已。一般我们使用注解的方式使用AOP就好了。
- 注解的方式使用Spring AOP就了解几个切点表达式，几个增强/通知的注解就完事了，是不是贼简单...使用XML的方式和注解其实没有很大的区别，很快就可以上手啦。
- 引介/引入切面也算是一个比较亮的地方，可以用代理的方式为某个对象实现接口，从而能够使用借口下的方法。这种方式是非侵入式的~
- 要增强的方法还可以接收与被代理方法一样的参数、绑定被代理方法的返回值这些功能...

最后，将我们上一次IOC的思维导图补充AOP的知识点上去吧~~~



![img](assets/163925ac9488dccf)

# 参考 #

1. [Spring【AOP模块】就是这么简单](<https://juejin.im/post/5aa8edf06fb9a028d0432584>)
