---
title: java并发（二）：线程安全性
type: tags
tags:
  - null
date: 2019-02-28 20:07:37
categories:
description:
---

# 线程安全性

当多个线程访问某个类时，不管运行时环境采用**何种调度方式**或者这些进程将如何交替执行，并且在主调代码中不需要**任何额外的同步或协同**，这个类都能表现出**正确的行为**。

**三个方面**

- 原子性：提供了互斥访问，同一时刻只能有一个线程对它进行操作
- 可见性：一个线程对主内存的修改可以及时被其他线程观察到
- 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般无序

# 原子性

提供了互斥访问，同一时刻只能有一个线程对它进行操作

## Atomic

JDK的Atomic包，通过CAS完成原子性

int类型变为AtomicInteger，自增的方法为incrementAndGet

### **CAS核心**

```Java
    //计数方法
    private static void add(){
        count.incrementAndGet();//先执行增加操作，再获取值
//        count.getAndIncrement(); //先获取当前的值，再执行增加操作
    }
```

源码实现，基于一个unsafe类

```Java
    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
```

以dowhile语句为核心实现,CompareAndSwap是C.A.S的核心.

因为使用循环，如果修改很频繁，会不断循环尝试修改，使得性能受到影响

```java
/**
 * var1:当前的对象
 */
public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //通过调用底层方法,获得var1对象底层当前的值
            //如果没有其他线程更改,则var5就会等于var2
            var5 = this.getIntVolatile(var1, var2);
            //比较底层的值var5与传入的值var2
            //如果底层的值与传入值相同,那么更新为var5+增量var4
            //如果值不相同,则将var2的值更改为底层当前的值,然后重新do,获得var5的值,进行比较
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

以native标识，即java底层的实现，不是java的实现

```java
    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

### 源码解析

**AtomicLong**

对于很精确的数值需要使用

**LongAdder**

原理：JVM对于普通的long与double，允许将64位的读操作与写操作拆分为两个32位的操作。

核心：将热点数据分离，将一个value分割为一个数组，每个线程针对一个数值，最后的value由数组的值合成，将单点的更新压力分散为多点的更新压力。在低并行的时候，对value直接更新。

优点：在高并发下，效率很高

缺点：如果有并行更新，可能导致统计数据有一些误差

**AtomicReference**

```Java
   private static AtomicReference<Integer> count = new AtomicReference<>(0);

//输出4
    public static void main(String[] args) {
        count.compareAndSet(0,2); // 2
        count.compareAndSet(0,1); // no
        count.compareAndSet(1,3); // no
        count.compareAndSet(2,4); // 4
        count.compareAndSet(3,5); // no
        log.info("count:{}",count.get());
    }
```

**AtomicReferenceFieldUpdater**

以**原子性更新**某个类的一个实例的一个字段，字段必须volatile，并且非static

```Java
    private static AtomicIntegerFieldUpdater<AtomicExample5> updater = AtomicIntegerFieldUpdater.newUpdater(AtomicExample5.class,"count");

    @Getter
    public volatile int count = 100;

    private static AtomicExample5 atomicExample5 = new AtomicExample5();

    public static void main(String[] args) {
        if (updater.compareAndSet(atomicExample5,100,120)){
            log.info("update success,{}",atomicExample5.getCount());
        }
        if (updater.compareAndSet(atomicExample5,100,120)){
            log.info("update success,{}",atomicExample5.getCount());
        }else {
            log.error("update error{}",atomicExample5.getCount());
        }


    }
```

**AtomicLongArray**

更新一个long的数组

**AtomicBoolean**

实现代码只执行一次

```Java
   private static AtomicBoolean isHappend = new AtomicBoolean(false);
   private static void test(){
        if (isHappend.compareAndSet(false, true)) {

            log.info("execute");
        }
    }
```



### CAS的ABA问题

ABA问题：在CAS操作中，其他线程将数据A改为B，又改为A。

解决：在每次更新的时候，记录一个版本号，每次更新+1

通过AtomicStampReference实现，核心方法为CompareAndSet

## 原子性：锁

synchronized：依赖JVM，在该关键字作用对象的作用范围内，只有一个线程可以操作

Lock：依赖特殊的CPU指令，由代码实现。ReentrantLock

### synchronized

同步锁，修饰的对象：

- 修饰代码块：大括号括起来的代码，作用于**调用的对象**
- 修饰方法：整个方法，作用于**调用的对象**
- 修饰静态方法：整个静态方法，作用于**所有对象**
- 修饰类：括号括起来部分，作用于**所有对象**

作用于所有的对象。则如果两个对象调用一个修饰的方法，他们会并行执行。而如果调用一个静态方法，则他们无法并行执行。

对于修饰代码块与修饰方法，不同的调用对象互相不影响

**继承**

如果子类调用继承于父类的synchronize方法（synchronize不属于一个类），是没有synchronize效果的，必须显式声明

### Lock

## 原子性对比：

- synchronize；不可中断锁，适合竞争不激烈，可读性好
- Lock：可中断锁，多样化同步，竞争激烈能维持常态
- Atomic：竞争激烈能维持常态，比lock性能好，但是只能同步一个值

# 可见性

一个线程对主内存的修改可以及时被其他线程观察到。

导致共享变量在线程间不可见的原因

- 线程交叉执行
- 重排序结合线程交叉执行
- 共享变量更新后的值没有在工作内存与主内存间及时更新

## synchronized

JVM对synchronize的规定

- 线程解锁前，必须把共享变量的最新值刷新到主内存
- 线程加锁时，将清空工作内存中共享变量的值，从而使用共享内存是需要从主内存中重新读取最新的值

## volatile

通过加入内存屏障与禁止重排序优化来实现。

- 对volatile变量**写操作**时，会在写操作后加入一条store屏障指令，将本地内存中的共享变量值刷新到主内存。（每次写之后都刷新），CPU指令级别进行操作。

![1551425245832](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551425245832.png)

- 对volatile变量**读操作**时，会在读操作前加入一条load屏障指令，从主内存中读取共享变量。（每次读从主内存读）

![1551425259624](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551425259624.png)

但是volatile**无法保证**线程安全。

```java
count++;
// 1、count//获得的是最新值
// 2、+1   //两个进程同时++
// 3、count//同时写回,即丢掉了一次.
```

volatile**使用**

- 对变量的写操作不依赖于当前值
- 该变量没有包含在具有其他变量的不变式中。

可以被写入 volatile 变量的这些有效值独立于任何程序的状态，包括变量的当前状态。

使用场景

- 很适合作为状态标示量
- 检查两次

# 有序性

一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般无序

方式：volatile、synchronized、lock、java内存模型的先天有序性（happens before原则）

## happens before原则

- 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
- 锁定操作：一个unlock操作先行发生于后面对同一个锁的lock操作
- volatile变量规则：对一个变量的写操作先行发生于后面对于中国变量的读操作
- 传递规则；如果操作A先行发生于操作B，而操作B又先行发生于操作C，则操作A先行发生于操作C
- 线程启动规则：Thread对象的start（）方法先行发生于此线程的每一个动作
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测
- 对象终结规则：一个对象的初始化完成先行发生于它的finalize（）方法的开始

如果两个操作的执行次序无法从happens before推导出来，则JVM可以对它进行随意的重排序。

# 总结

原子性：Atomic包、CAS算法、synchronized、Lock

可见性：synchronized、volatile

有序性：happens-before

# 参考 #

1. [慕课网《Java并发编程入门与高并发面试 》课程学习](https://github.com/youzhidakeai/concurrency)
