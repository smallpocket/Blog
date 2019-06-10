---
title: java并发（2）：安全发布对象
type: tags
tags:
  - 并发编程
date: 2019-03-01 15:42:47
categories: Java
description:
---

# 发布对象与逸出 

发布对象：发布一个对象指它能够被当前范围以外的代码所使用。发布一个对象，同时将发布该对象所有的非私有域引用的对象。

```java
public class UnsafePublish {

    private String[] states = {"a","b","c"};

    /**
     * 直接获得了私有对象states的引用
     * @return
     */
    public String[] getStates(){
        return states;
    }

    public static void main(String[] args){
        UnsafePublish unsafePublish = new UnsafePublish();
        log.info("{}", Arrays.toString(unsafePublish.getStates()));
        //发布states对象,无法确定其他线程是否会修改该对象的数据
        //因此在使用这个对象的数据的时候,无法完全确定对象里面的数据
        //即线程不安全的
        unsafePublish.getStates()[0] = "d";
        log.info("{}", Arrays.toString(unsafePublish.getStates()));

    }
}
```

对象逸出：一个对象在尚未准备好的时候就发布，使得它被其他线程可见

逸出：

- this引用在构造期间逸出，即对象在没有通过构造函数构造完毕（执行到了构造函数的某一句）时候逸出。
- 当对象在构造函数当中创建一个线程，this引用总是被新线程共享.

如果要在构造函数中创建线程

- 使用工厂方法或者私有构造函数来完成

```Java
public class Escape {

    private int thisCanBeEscape = 0;

    public Escape(){
        //在这里启动了一个线程,新线程已经可以看到escape类的对象
        new InnerClass();
    }

    public class InnerClass{
        public InnerClass(){
            //包含了对封装实例隐含的引用,在对象未构造完全就被发布
            
            log.info("{}",Escape.this.thisCanBeEscape);
        }
    }

    public static void main(String[] args) {
        new Escape();
    }
}

```

# 安全发布对象的四种方法

- 通过静态初始化器初始化对象的引用（JVM内部的同步机制）
- 将它的引用存储到volatile域或者atomicReference对象中
- 将它的引用存储到正确创建的对象的final域
- 或者将它的引用存储到由锁正确保护的域中

## 单例发布对象

### 懒汉式

#### 线程不安全法

```Java
/**
 * 懒汉模式
 * 单例实例在第一次使用时候进行创建
 */
@NotRecommend
public class SingletonExample1 {

    //私有的构造函数
    //即其他途径无法创建这个类的对象
    private SingletonExample1(){
        //包含对资源的处理等等
    }

    //单例对象
    private static SingletonExample1 instance = null;

    //静态的工厂方法
    private static SingletonExample1 getInstance(){
        //多线程环境很容易出现问题
        if (instance == null){
            instance = new SingletonExample1();
        }
        return instance;
    }
}
```

**双重检测机制**(线程不安全)

```Java
/**
 * 懒汉模式 --> 双重同步锁单例模式
 * 单例实例在第一次使用时候进行创建
 */
@NotRecommend
public class SingletonExample4 {

    //私有的构造函数
    private SingletonExample4(){}

    // 1.memory = allocate() 分配对象的内存空间
    // 2.ctorInstance() 初始化对象
    // 3. instance = memory 设置instance 指向刚分配的内存

    //JVM和CPU优化，发生了指令重排

    // 1.memory = allocate() 分配对象的内存空间
    // 3. instance = memory 设置instance 指向刚分配的内存
    // 2.ctorInstance() 初始化对象

    //在第三步的时候,instance!=null
    //而此时在指令重排下,另一个线程就会获得一个没有初始化对象的引用,并将其返回

    //单例对象
    private static SingletonExample4 instance = null;

    //静态的工厂方法
    private static SingletonExample4 getInstance(){
        if (instance == null){ //双重检测机制          //B
            synchronized (SingletonExample4.class){ //同步锁
                if (instance == null){
                    instance = new SingletonExample4(); //A - 3
                }
            }
        }
        return instance;
    }
```

#### **线程安全法**

不推荐法

```Java
public class SingletonExample3 {

    //私有的构造函数
    private SingletonExample3(){}

    //单例对象
    private static SingletonExample3 instance = null;

    //静态的工厂方法
    //synchronized限制,而存在性能开销
    private static synchronized SingletonExample3 getInstance(){
        if (instance == null){
            instance = new SingletonExample3();
        }
        return instance;
    }
}
```

**双重同步锁**

基于volatile

```java
/**
 * 懒汉模式 --> 双重同步锁单例模式
 * 单例实例在第一次使用时候进行创建
 */
@ThreadSafe
public class SingletonExample5 {

    //私有的构造函数
    private SingletonExample5(){}

    // 1.memory = allocate() 分配对象的内存空间
    // 2.ctorInstance() 初始化对象
    // 3. instance = memory 设置instance 指向刚分配的内存

    //单例对象 volatitle+ 双重检测机制 -> 禁止指令重排序
    private volatile static SingletonExample5 instance = null;

    //静态的工厂方法
    private static SingletonExample5 getInstance(){
        if (instance == null){ //双重检测机制          //B
            synchronized (SingletonExample5.class){ //同步锁
                if (instance == null){
                    instance = new SingletonExample5(); //A - 3
                }
            }
        }
        return instance;
    }
}

```

### 饿汉模式

通过静态域实现

```Java
/**
 * 饿汉模式
 * 单例实例在装载使用时候进行创建
 */
@ThreadSafe
public class SingletonExample2 {

    //私有的构造函数
    private SingletonExample2(){
        //如果构造方法中存在过多的功能,则在加载时会过慢,存在性能问题
        //只进行资源加载而没有实际调用,则会导致资源浪费
    }

    //单例对象
    private static SingletonExample2 instance = new SingletonExample2();

    //静态的工厂方法
    private static SingletonExample2 getInstance(){
        return instance;
    }
}

```

通过静态块实现

```
@ThreadSafe
public class SingletonExample6 {

    //私有的构造函数
    private SingletonExample6(){}
	//静态资源是顺序执行的
    //单例对象
    private static SingletonExample6 instance = null;
	//必须写在后面,如果写在前面,则会被上一句赋值为null
    static {
        instance = new SingletonExample6();
    }

    //静态的工厂方法
    private static SingletonExample6 getInstance(){
        return instance;
    }

    public static void main(String[] args) {
        System.out.println(getInstance().hashCode());
        System.out.println(getInstance().hashCode());

    }
}
```

### 枚举模式

```Java
/**
 * 枚举模式：最安全
 */
@ThreadSafe
@Recommend
public class SingletonExample7 {
    //私有的构造函数
    private SingletonExample7(){}

    public static SingletonExample7 getInstance(){
        //在实际使用的时候才会初始化
        return Singleton.INSTANCE.getSingleton();
    }

    private enum Singleton{
        INSTANCE;
        private SingletonExample7 singleton;

        //JVM保证这个方法绝对只调用一次
        Singleton(){
            singleton = new SingletonExample7();
        }

        public SingletonExample7 getSingleton(){
            return singleton;
        }
    }
}
```



# 参考 #

1. 
