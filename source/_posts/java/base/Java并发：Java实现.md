---
title: Java并发：Java实现
type: tags
tags:
  - 并发编程
date: 2019-03-05 19:31:00
categories: Java
description:
---

# 基本的线程机制

## 定义任务

有三种使用线程的方法：

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

### Runnable接口

线程可以驱动任务，因此需要一种描述任务的方式。

通过Runnable接口提供，只需要实现接口，并编写run方法。但是该方法并不产生任何内在的线程能力。

要实现线程行为，必须显式地将一个任务附着到线程上，通过Thread的start方法启动。

```Java
//继承接口
public class LiftOff implements Runnable {
  protected int countDown = 10; // Default
  private static int taskCount = 0;
  private final int id = taskCount++;
  public LiftOff() {}
    
  public LiftOff(int countDown) {
    this.countDown = countDown;
  }
    
  public String status() {
    return "#" + id + "(" + (countDown > 0 ? countDown : "Liftoff!") + "), ";
  }
    
  //实现run方法
  public void run() {
    while(countDown-- > 0) {
      System.out.print(status());
        
      Thread.yield();
    }
  }
} ///:~
```

### Callable 从任务中返回值

**Runable与Callable**

- Runnable是执行工作的独立任务，但是不返回任何值
  - 使用ExecutorService.execute()调用
- Callable在完成任务的时候，可以返回一个值
  - 使用ExecutorService.sumbit()调用
  - 返回的对象是Future对象
    - isDone方法查询是否完成，任务完成时具有结果
    - get方法会获取结果
      - 如果没有结果，则会阻塞，直至结果就绪

Callable是一种具有类型参数的泛型，它的类型参数表示从方法call()当中返回的值

```Java
class TaskWithResult implements Callable<String> {
  private int id;
    
  public TaskWithResult(int id) {
    this.id = id;
  }
    
  //执行的方法  
  public String call() {
    return "result of TaskWithResult " + id;
  }
}

public class CallableDemo {
  public static void main(String[] args) {
    ExecutorService exec = Executors.newCachedThreadPool();
    ArrayList<Future<String>> results =
      new ArrayList<Future<String>>();
    for(int i = 0; i < 10; i++)
      results.add(exec.submit(new TaskWithResult(i)));
    for(Future<String> fs : results)
      try {
          //get方法会阻塞，直至产生结果
        // get() blocks until completion:
        System.out.println(fs.get());
      } catch(InterruptedException e) {
        System.out.println(e);
        return;
      } catch(ExecutionException e) {
        System.out.println(e);
      } finally {
        exec.shutdown();
      }
  }
} /* Output:
result of TaskWithResult 0
result of TaskWithResult 1
result of TaskWithResult 2
result of TaskWithResult 3
result of TaskWithResult 4
result of TaskWithResult 5
result of TaskWithResult 6
result of TaskWithResult 7
result of TaskWithResult 8
result of TaskWithResult 9
*///:~

```

### 继承Thread

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

```Java
public class MyThread extends Thread {
    public void run() {
        // ...
    }
}
public static void main(String[] args) {
    MyThread mt = new MyThread();
    mt.start();
}
```

## Thread类

将Runnable对象转变为工作任务的传统方式，是将它提交给一个Thread构造器

从输出中可以看到，start方法很快就返回了，是先于run方法执行的

```Java
public class BasicThreads {
  public static void main(String[] args) {
      //构造器需要一个runnable对象
    Thread t = new Thread(new LiftOff());
      //start方法为该线程执行必要的初始化操作
      //调用runnable的run方法，启动任务
    t.start();
    System.out.println("Waiting for LiftOff");
  }
}
/* Output: (90% match)
Waiting for LiftOff
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!),
*///:~
```

**生命周期**

main函数创建Thread时，没有捕获这些对象的引用。但是每个Thread都**注册**了它自己，因此确实存在一个对它的引用，而且在它的任务退出run且死亡前，垃圾回收器无法清除它。因此一个线程会创建一个单独的执行线程，在对start调用完成后，它仍旧会继续存在。



### 休眠

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

```Java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```



### 优先级

线程的优先级将该线程的重要性传递给了调度器。调度器倾向于让优先级最高的线程执行。（通常不要设置）

```java
getPriority()//获取当前线程优先级
setPriority()//修改优先级	
```

### 让步Thread.yield()

静态方法Thread.yield()的调用

- 是对线程调度器（Java线程机制的一部分，可以将CPU从一个线程转移给另一个线程）的一种建议
- 在声明：我已经执行完生命周期中最重要的部分，现在是切换给其他任务执行一段时间的大好时机

当一件完成了run中一次工作，可以给线程调度机制一个暗示，即可以让别的线程使用CPU了。但是，**这只是一个暗示，没有任何机制保证它将会被采纳**。因此在重要的控制中，不能依赖yield

```Java
public void run() {
    Thread.yield();
}
```



### 加入一个线程

join()方法

- 一个线程可以在其他线程上调用join()方法
  - 等待一段时间，直到第二个线程结束才继续执行
  - 如果某个线程在另一个线程t上调用t.join()，此线程将被挂起，直到t结束才恢复
  - 或者为join添加一个超时参数
  - 对join的调用可以被中断，在调用线程上调用interrupt()方法（try -catch）

### 后台线程

指在程序运行的时候，在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分，

当所有的非后台线程结束时，程序就会终止，同时杀死进程所有后台线程。只要存在非后台线程，程序就不会终止。

在线程启动前调用setDaemon()方法，设置为后台线程。main属于非后台线程。

```Java
public void run() {
    Thread.yield();
}
```



### 线程组

线程组持有一个线程集合

> 是一次不成功的尝试，忽略即可

## Executor

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

java.util.concurrent包当中的执行器（excutor）帮助管理Thread对象。

```Java
public class CachedThreadPool {
  public static void main(String[] args) {
      //Executors.newFixedThreadPool(15);则可以预先进行现场分配，固定现场池大小
      //newCachedThreadPool是首选，会创建与所需数量相同的线程，在它回收旧线程时停止创建新线程
      //SingleThreadExecutor是数量为1的线程池，用于在一个线程中连续运行任何事物（长期存活的任务），例如监听进入的套接字连接
    ExecutorService exec = Executors.newCachedThreadPool();
    for(int i = 0; i < 5; i++)
      exec.execute(new LiftOff());
      //该方法防止新任务提交给executor
      //将继续运行已经提交过的任务，并且程序在所有任务完成后尽快推出
    exec.shutdown();
  }
} /* Output: (Sample)
#0(9), #0(8), #1(9), #2(9), #3(9), #4(9), #0(7), #1(8), #2(8), #3(8), #4(8), #0(6), #1(7), #2(7), #3(7), #4(7), #0(5), #1(6), #2(6), #3(6), #4(6), #0(4), #1(5), #2(5), #3(5), #4(5), #0(3), #1(4), #2(4), #3(4), #4(4), #0(2), #1(3), #2(3), #3(3), #4(3), #0(1), #1(2), #2(2), #3(2), #4(2), #0(Liftoff!), #1(1), #2(1), #3(1), #4(1), #1(Liftoff!), #2(Liftoff!), #3(Liftoff!), #4(Liftoff!),
*///:~
```

### 捕获异常

由于线程的本质特性，使得你**不能捕获从线程中逃逸的异常**。一旦逃逸出任务的run()方法，就会向外传播到控制台，除非采用**特殊的步骤**进行捕获。

Executor可以解决这个问题

修改产生线程的方式，Thread.UncaughtExceptionHandler是一个新的接口，允许在每个thread对象上附着一个异常处理器。



## 任务的多种启动

任务类的实现方式：

- 实现runnable或callable
- 从thread继承

```Java
public class SimpleThread extends Thread {
  private int countDown = 5;
  private static int threadCount = 0;
  public SimpleThread() {
    // Store the thread name:
    super(Integer.toString(++threadCount));
    start();
  }
  public String toString() {
    return "#" + getName() + "(" + countDown + "), ";
  }
  public void run() {
    while(true) {
      System.out.print(this);
      if(--countDown == 0)
        return;
    }
  }
  public static void main(String[] args) {
    for(int i = 0; i < 5; i++)
      new SimpleThread();
  }
} /* Output:
#1(5), #1(4), #1(3), #1(2), #1(1), #2(5), #2(4), #2(3), #2(2), #2(1), #3(5), #3(4), #3(3), #3(2), #3(1), #4(5), #4(4), #4(3), #4(2), #4(1), #5(5), #5(4), #5(3), #5(2), #5(1),
*///:~
```

- 自管理的runnable

```java
public class SelfManaged implements Runnable {
  private int countDown = 5;
    //定义了一个thread，并以对象本身进行初始化
  private Thread t = new Thread(this);
    //在构造器当中，进行线程启动
    //但可能不处于不稳定状态，因为在构造器当中使用线程
  public SelfManaged() { t.start(); }
  public String toString() {
    return Thread.currentThread().getName() +
      "(" + countDown + "), ";
  }
  public void run() {
    while(true) {
      System.out.print(this);
      if(--countDown == 0)
        return;
    }
  }
  public static void main(String[] args) {
    for(int i = 0; i < 5; i++)
      new SelfManaged();
  }
} /* Output:
Thread-0(5), Thread-0(4), Thread-0(3), Thread-0(2), Thread-0(1), Thread-1(5), Thread-1(4), Thread-1(3), Thread-1(2), Thread-1(1), Thread-2(5), Thread-2(4), Thread-2(3), Thread-2(2), Thread-2(1), Thread-3(5), Thread-3(4), Thread-3(3), Thread-3(2), Thread-3(1), Thread-4(5), Thread-4(4), Thread-4(3), Thread-4(2), Thread-4(1),
*///:~
```

- 使用内部类将线程代码隐藏在类中

## 术语

任务与线程是相互分离的

- 执行的任务
  - Runnable
- 驱动任务的线程
  - 对于线程Thread类无实际的控制权
  - 将一个线程附着到任务上，以使得这个线程可以驱动任务

# 共享受限资源

两个线程试图同时使用一个资源。如在一个地方停车。

## 解决共享资源竞争

永远不知道一个线程何时运行

解决冲突的方法：在资源被一个任务使用时，在其上加锁。

**采用序列化访问共享资源**的方案解决线程冲突问题，在给定时刻只允许一个任务访问共享资源。

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

### synchronized

java采用synchronized关键字提供锁，当任务执行该关键字保护的片段时

流程

- 检查锁是否可用
- 可用

  - 获取锁，执行代码

  - 释放锁
- 不可用
  - 阻塞，直至锁释放

synchronized是可重入锁，即同一个对象可以重复获得该方法的锁。

- 只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

**实现**

- 同步代码块

```Java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```Java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

```Java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

- 同步方法

```Java
public synchronized void func () {
    // ...
}
```

它和同步代码块一样，作用于同一个对象。

- 同步类

```Java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

- 同步静态方法

```Java
public synchronized static void fun() {
    // ...
}
```

作用于整个类。

### Lock

- 一种显式的互斥机制
- Lock对象必须被**显式地创建、锁定和释放**。
- 因此相比于内建的锁形式相比，代码缺乏优雅性。
- 对于解决某些类型问题，更加灵活

**锁的使用**

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

```Java
public class LockExample {

    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁。
        }
    }
}
public static void main(String[] args) {
    LockExample lockExample = new LockExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> lockExample.func());
    executorService.execute(() -> lockExample.func());
}
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```



```Java
//ReentrantLock允许尝试着获取但最终未获取锁
public class AttemptLocking {
  private ReentrantLock lock = new ReentrantLock();
  //如果其他人已经获取了锁，那么你就可以决定离开去执行其他一些事情，而不是等待直至这个锁被释放
    public void untimed() {
      boolean captured = lock.tryLock();
      try {
        System.out.println("tryLock(): " + captured);
      } finally {
        if(captured)
          lock.unlock();
      }
  }
    //尝试去获取锁，该尝试可以在2s后失败
  public void timed() {
    boolean captured = false;
    try {
      captured = lock.tryLock(2, TimeUnit.SECONDS);
    } catch(InterruptedException e) {
      throw new RuntimeException(e);
    }
    try {
      System.out.println("tryLock(2, TimeUnit.SECONDS): " +
        captured);
    } finally {
      if(captured)
        lock.unlock();
    }
  }
  public static void main(String[] args) {
    final AttemptLocking al = new AttemptLocking();
    al.untimed(); // True -- lock is available
    al.timed();   // True -- lock is available
    // Now create a separate task to grab the lock:
    new Thread() {
      { setDaemon(true); }
      public void run() {
        al.lock.lock();
        System.out.println("acquired");
      }
    }.start();
    Thread.yield(); // Give the 2nd task a chance
    al.untimed(); // False -- lock grabbed by task
    al.timed();   // False -- lock grabbed by task
  }
} /* Output:
tryLock(): true
tryLock(2, TimeUnit.SECONDS): true
acquired
tryLock(): false
tryLock(2, TimeUnit.SECONDS): false
*///:~
```

### Lock与synchronized比较

> 除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。



- 锁的实现

  - synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

- 性能

  - 新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

- 错误处理

  - 使用synchronized，某些事物失败了，就会抛出异常，无法去做任何清理工作，维护系统处于良好
  - Lock对象可以使用finally子句将系统维护在正确状态

- 代码量

  - synchronized代码量较少，出现错误可能性较低

- 等待可中断

  - 当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

  - ReentrantLock 可中断，而 synchronized 不行。

- 公平锁

  - 公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

  - synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

- 锁绑定多个条件

  - 一个 ReentrantLock 可以同时绑定多个 Condition 对象。

- 适用场景
  - synchronized不能尝试获取锁，且最终获取锁会失败，或者尝试获取锁一段时间，然后放弃它
  - Lock对于锁具有更细力度的控制力，对于实现专有同步结构是有效的



### 何时进行同步

> 如果你正在写一个变量，它可能接下来将被另一个线程读取，或者正在读取一个上一次已经被另一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须用系统的监视器锁同步

## 原子性与易变性

原子操作：不能被线程调度机制中断的操作。一旦操作开始，那么它一定可以在可能发生的“上下文切换”前执行完毕

> 原子操作需要进行同步控制

原子性可以应用于除long和double之外所有基本类型上的“简单操作”，对于这些操作可以保证它们会被当成原子操作来操作内存。

对于64位的long与double进行读与写操作会被JVM当做两个分离的32位操作，因此可能会出现上下文切换。

### 可见性

一个任务做出的修改，即使在不中断的意义上讲是原子性的，对于其他任务也可能是不可视的。

- 修改可能只是暂时地存储在本地处理器的缓存时

因此不同的任务对于应用的状态有不同的视图。

### volatile

保证了应用的可视性，如果一个域为volatile的，那么只要对这个域进行了写操作，所有的读操作就都可以看到这个修改。

如果多个任务在同时访问某个域，那么这个域就应该是volatile的，否则这个域就只能由同步来访问。

如果一个域定义为volatile的，那么就会告知编译器不要只想任何一处读取和写入操作的优化。目的是用线程中的局部变量维护对这个域的精确同步。

**意外**

volatile无法工作的情况

- 如果一个域的值依赖于它之前的值
- 如果域的值受到其他域的值现在

## 原子类

AtomicInteger、AtomicLong、AtomicReference等原子性变量类

### CAS操作

原子性更新操作CAS

```java
boolean compareAndSet(expectedValue，updateValue)；
```

## 临界区

同步控制块

希望防止多个线程同时访问方法内部的部分代码而不是防止访问整个方法，通过这种方式分离出来的代码段被称为**临界区**

```Java
synchronized（syncObjet）{
    
}
```

## 在其他对象上同步

synchronized块需要给定一个在其上进行同步的对象，使用synchronized（this）会使得该对象其他的synchronized方法无法被调用。

如果必须在另一个对象上进行同步，则必须确保所有相关的任务都在同一个对象上同步。

```Java
class DualSynch {
  private Object syncObject = new Object();
  public synchronized void f() {
    for(int i = 0; i < 5; i++) {
      print("f()");
      Thread.yield();
    }
  }
  public void g() {
    synchronized(syncObject) {
      for(int i = 0; i < 5; i++) {
        print("g()");
        Thread.yield();
      }
    }
  }
}
//两个任务同时进入了同时一个对象
//对象上的方法是在不同的锁上进行同步
public class SyncObject {
  public static void main(String[] args) {
    final DualSynch ds = new DualSynch();
    new Thread() {
      public void run() {
        ds.f();
      }
    }.start();
    ds.g();
  }
} 
```

## 线程本地存储

- 根除对变量的共享而防止任务在共享资源上产生冲突。
- 通过为使用相同变量的每个不同的线程都创建不同的存储
- Global 意思是在当前线程中，任何一个点都可以访问到ThreadLocal的值。
- Local 意思是该线程的ThreadLocal只能被该线程访问，一般情况下其他线程访问不到。

### ThreadLocal

为变量生成新的存储，并将状态与线程关联起来

```Java
class Accessor implements Runnable {
  private final int id;
  public Accessor(int idn) { id = idn; }
  public void run() {
    while(!Thread.currentThread().isInterrupted()) {
      ThreadLocalVariableHolder.increment();
      System.out.println(this);
      Thread.yield();
    }
  }
  public String toString() {
    return "#" + id + ": " +
      ThreadLocalVariableHolder.get();
  }
}

public class ThreadLocalVariableHolder {
  private static ThreadLocal<Integer> value =
    new ThreadLocal<Integer>() {
      private Random rand = new Random(47);
      protected synchronized Integer initialValue() {
        return rand.nextInt(10000);
      }
    };
  public static void increment() {
    value.set(value.get() + 1);
  }
  public static int get() { return value.get(); }
  public static void main(String[] args) throws Exception {
    ExecutorService exec = Executors.newCachedThreadPool();
    for(int i = 0; i < 5; i++)
      exec.execute(new Accessor(i));
    TimeUnit.SECONDS.sleep(3);  // Run for a while
    exec.shutdownNow();         // All Accessors will quit
  }
}
/* Output: (Sample)
#0: 9259
#1: 556
#2: 6694
#3: 1862
#4: 962
#0: 9260
#1: 557
#2: 6695
#3: 1863
#4: 963
...
*///:~
```

# 终结任务

某些情况下，任务必须更加突然地终止

方法

- 在程序当中设置一个标志位，在程序的某一点当中检查标志，以决定是否跳出循环

## 线程状态

- 新建
- 就绪
- 阻塞
  - 一个任务进入阻塞状态的原因
    - 调用sleep()方法
    - 调用wait()使得线程挂起，直到线程得到了notify()或notifyAll()，任务才会进入就绪
    - 任务在等待某个输入、输出完成
    - 任务视图在某个对象上调用其同步方法，但是对象锁不可用
- 死亡

## 在阻塞时终结

对于处于阻塞状态的任务，不能等待其到达代码中可以检查状态值的某一点，因而决定让它主动终止，就必须强制这个任务跳出阻塞

## 中断

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

打断被阻塞的任务，可能需要清理资源，因此类似于抛出异常。

### interrupt

Thread类的一个方法，终止被阻塞的任务。

这个方法将设置线程的中断状态

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

```Java
public class InterruptExample {

    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()) {
                // ..
            }
            System.out.println("Thread end");
        }
    }
}
public static void main(String[] args) throws InterruptedException {
    Thread thread2 = new MyThread2();
    thread2.start();
    thread2.interrupt();
}
Thread end
```

### InterruptedException

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```Java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new MyThread1();
    thread1.start();
    thread1.interrupt();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

### 被互斥所阻塞

试图在一个对象上调用synchronized方法，而这个对象的锁已经被其他任务获得，那么调用任务将被挂起（阻塞），直至这个锁可获得

### Executor 的中断操作

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

```Java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> {
        try {
            Thread.sleep(2000);
            System.out.println("Thread run");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
    executorService.shutdownNow();
    System.out.println("Main run");
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:745)
```

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

```Java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```



## 检查中断



# 线程间协作

线程间相互协调，某些部分必须在其他部分被解决前解决。即存在前置条件

任务协作时，关键问题是这些任务间的握手，为了实现握手，使用了**互斥**。互斥确保只有一个任务可以响应某个信号，以消除任何困难的竞争条件。

在互斥上，为任务添加了一种途径，可以将自身挂起，直至某些外部条件发生变化。

## join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```Java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
A
B
```

## wait()与notifyAll()

**wait**：

- 表示等待某个条件发生变化，而改变这个条件超出了当前方法的控制能力，通常需要另一个任务来改变
- wait需要notify、notifyAll、或者令时间到期从而恢复执行
- wait期间，对象锁是释放的，当wait时，即在声明：我已经做完能做的所有事情，因此等待，并且希望其他的synchronized操作在条件合适情况下执行
  - sleep方法不释放对象的锁
- 需要用一个检查感兴趣的条件的while循环包围wait。需要检查锁感兴趣的特定条件，在不满足条件下重新wait
  - 可能有多个任务出于相同原因等待同一个锁，而第一个唤醒任务可能改变状态，此时应该再次通过调用wait以重新挂起
  - 在这个任务从其wait被唤醒的时刻，可能有其他的任务已经做出改变，使得这个任务此时不能执行。此时应该再次调用wait以挂起
  - 任务出于不同的原因在等待你的对象上的锁，需要检查是否已经由正确的原因唤醒，如果不是，再次调用wait
- 只能用在同步方法或者同步控制块中使用
  - 如果在非同步控制块中，可以通过编译，但如果任务（线程）没有持有对象的锁，则会抛出异常 IllegalMonitorStateException。

```Java
public class WaitNotifyExample {

    public synchronized void before() {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    WaitNotifyExample example = new WaitNotifyExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
before
after
```

**wait与sleep**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

**notify**

- 唤醒wait调用而被挂起的任务
- 任务首先获取当它进入wait时释放的锁
  - 锁不可用，则继续挂起
  - 锁可用，并获取，任务被唤醒

**调用区域**

wait与notify是Object对象的一部分，因为这些方法操作的锁也是所有对象的一部分，因此他们不是Thread类中。

方法只能再同步控制块里进行调用，如果在非同步控制块中，可以通过编译，但如果任务（线程）没有持有对象的锁，则会抛出异常

```Java
class Car {
  private boolean waxOn = false;
  public synchronized void waxed() {
    waxOn = true; // Ready to buff
    notifyAll();
  }
  public synchronized void buffed() {
    waxOn = false; // Ready for another coat of wax
    notifyAll();
  }
  public synchronized void waitForWaxing()
  throws InterruptedException {
    while(waxOn == false)
      wait();
  }
  public synchronized void waitForBuffing()
  throws InterruptedException {
    while(waxOn == true)
      wait();
  }
}

class WaxOn implements Runnable {
  private Car car;
  public WaxOn(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        printnb("Wax On! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.waxed();
        car.waitForBuffing();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax On task");
  }
}

class WaxOff implements Runnable {
  private Car car;
  public WaxOff(Car c) { car = c; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        car.waitForWaxing();
        printnb("Wax Off! ");
        TimeUnit.MILLISECONDS.sleep(200);
        car.buffed();
      }
    } catch(InterruptedException e) {
      print("Exiting via interrupt");
    }
    print("Ending Wax Off task");
  }
}

public class WaxOMatic {
  public static void main(String[] args) throws Exception {
    Car car = new Car();
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new WaxOff(car));
    exec.execute(new WaxOn(car));
    TimeUnit.SECONDS.sleep(5); // Run for a while...
    exec.shutdownNow(); // Interrupt all tasks
  }
}
```

**错失的信号**

- 假设T2获得someCondition的值为true
- 而此时调度器切换到T1，T1执行notify
- T2继续执行，进行wait，notify已经错失，进入死锁

```Java
T1:
synchronized（sharedMonitor）{
	//防止T2调用wait的动作
    <setup condition for T2>
    sharedMonitor.notify();
}
T2:
while(someCondition){
	//point1
	synchronized(sharedMonitor){
        sharedMonitor.wait();
	}
}
//消除竞争条件
while(someCondition){
	//point1
	synchronized(sharedMonitor){
	while(someCondition){
        sharedMonitor.wait();
    }
}
```

## notify()与notifyAll()

使用notify的条件

- 使用notify而不是notifyAll是一种优化
- 使用notify，则必须保证被唤醒的是恰当的任务
- 使用notify则所有任务必须等待相同的条件
- 如果有多个任务在等待不同的条件，则不知道是否唤醒了恰当的任务
- 使用notify则条件变化时，必须只有一个任务从中受益
- 这些限制对所有可能存在的子类都必须总是起作用的

**否则就必须使用**notifyAll

notifyAll

- 当notifyAll因某个特定锁而被调用时，只有等待**这个锁**的任务才会被唤醒
- 因为notifyAll语句是需要在一个同步控制块里面的，所以也清楚是哪个锁

## await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

``` java
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
before
after
```



## 生产者与消费者

任务协作的模型：

- 生产者
  - 厨师，准备膳食，在准备好之后，通知服务员
- 消费者
  - 服务员，等待厨师准备膳食
  - 接到通知，上菜，返回继续等待

```Java
class Meal {
  private final int orderNum;
  public Meal(int orderNum) { this.orderNum = orderNum; }
  public String toString() { return "Meal " + orderNum; }
}
//消费者
class WaitPerson implements Runnable {
  private Restaurant restaurant;
  public WaitPerson(Restaurant r) { restaurant = r; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        synchronized(this) {
          while(restaurant.meal == null)
            wait(); // ... for the chef to produce a meal
        }
        print("Waitperson got " + restaurant.meal);
          //在消费者取走的动作当中，防止生产者写入数据
        synchronized(restaurant.chef) {
          restaurant.meal = null;
          restaurant.chef.notifyAll(); // Ready for another
        }
      }
    } catch(InterruptedException e) {
      print("WaitPerson interrupted");
    }
  }
}
//生产者
class Chef implements Runnable {
  private Restaurant restaurant;
  private int count = 0;
  public Chef(Restaurant r) { restaurant = r; }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        synchronized(this) {
          while(restaurant.meal != null)
            wait(); // ... for the meal to be taken
        }
        if(++count == 10) {
          print("Out of food, closing");
          restaurant.exec.shutdownNow();
        }
        printnb("Order up! ");
        synchronized(restaurant.waitPerson) {
          restaurant.meal = new Meal(count);
          restaurant.waitPerson.notifyAll();
        }
        TimeUnit.MILLISECONDS.sleep(100);
      }
    } catch(InterruptedException e) {
      print("Chef interrupted");
    }
  }
}

public class Restaurant {
  Meal meal;
  ExecutorService exec = Executors.newCachedThreadPool();
  WaitPerson waitPerson = new WaitPerson(this);
  Chef chef = new Chef(this);
  public Restaurant() {
    exec.execute(chef);
    exec.execute(waitPerson);
  }
  public static void main(String[] args) {
    new Restaurant();
  }
} /* Output:
Order up! Waitperson got Meal 1
Order up! Waitperson got Meal 2
Order up! Waitperson got Meal 3
Order up! Waitperson got Meal 4
Order up! Waitperson got Meal 5
Order up! Waitperson got Meal 6
Order up! Waitperson got Meal 7
Order up! Waitperson got Meal 8
Order up! Waitperson got Meal 9
Out of food, closing
WaitPerson interrupted
Order up! Chef interrupted
*///:~

```

## 生产者-消费者与队列

基于同步队列来解决任务协作问题，同步队列在任何时刻只允许一个任务插入或移除数据

java.util.concurrent.（Linked/Array）BlockingQueue

- 当消费者任务试图从队列中获取对象，而此时队列为空，队列可挂起消费者
- 当有更多的元素可用，恢复消费者
- 相比于wait与notifyAll，简单并且可靠
- 解决了wait等存在的类之间的耦合

```Java
class LiftOffRunner implements Runnable {
    //一个blockingQueue
  private BlockingQueue<LiftOff> rockets;
  public LiftOffRunner(BlockingQueue<LiftOff> queue) {
    rockets = queue;
  }
  public void add(LiftOff lo) {
    try {
      rockets.put(lo);
    } catch(InterruptedException e) {
      print("Interrupted during put()");
    }
  }
  public void run() {
    try {
      while(!Thread.interrupted()) {
        LiftOff rocket = rockets.take();
        rocket.run(); // Use this thread
      }
    } catch(InterruptedException e) {
      print("Waking from take()");
    }
    print("Exiting LiftOffRunner");
  }
}

public class TestBlockingQueues {
  static void getkey() {
    try {
      // Compensate for Windows/Linux difference in the
      // length of the result produced by the Enter key:
      new BufferedReader(
        new InputStreamReader(System.in)).readLine();
    } catch(java.io.IOException e) {
      throw new RuntimeException(e);
    }
  }
  static void getkey(String message) {
    print(message);
    getkey();
  }
  static void
  test(String msg, BlockingQueue<LiftOff> queue) {
    print(msg);
    LiftOffRunner runner = new LiftOffRunner(queue);
    Thread t = new Thread(runner);
    t.start();
    for(int i = 0; i < 5; i++)
      runner.add(new LiftOff(5));
    getkey("Press 'Enter' (" + msg + ")");
    t.interrupt();
    print("Finished " + msg + " test");
  }
  public static void main(String[] args) {
    test("LinkedBlockingQueue", // Unlimited size
      new LinkedBlockingQueue<LiftOff>());
    test("ArrayBlockingQueue", // Fixed size
      new ArrayBlockingQueue<LiftOff>(3));
    test("SynchronousQueue", // Size of 1
      new SynchronousQueue<LiftOff>());
  }
} ///:~
```



## 任务间使用管道进行输入/输出

管道:

- PipedWriter允许任务向管道写
- PipedReader允许不同任务从同一个管道中读取

```Java
class Sender implements Runnable {
  private Random rand = new Random(47);
  private PipedWriter out = new PipedWriter();
  public PipedWriter getPipedWriter() { return out; }
  public void run() {
    try {
      while(true)
        for(char c = 'A'; c <= 'z'; c++) {
          out.write(c);
          TimeUnit.MILLISECONDS.sleep(rand.nextInt(500));
        }
    } catch(IOException e) {
      print(e + " Sender write exception");
    } catch(InterruptedException e) {
      print(e + " Sender sleep interrupted");
    }
  }
}

class Receiver implements Runnable {
  private PipedReader in;
  public Receiver(Sender sender) throws IOException {
    in = new PipedReader(sender.getPipedWriter());
  }
  public void run() {
    try {
      while(true) {
        // Blocks until characters are there:
          //如果没有数据,管道将阻塞
        printnb("Read: " + (char)in.read() + ", ");
      }
    } catch(IOException e) {
      print(e + " Receiver read exception");
    }
  }
}

public class PipedIO {
  public static void main(String[] args) throws Exception {
    Sender sender = new Sender();
    Receiver receiver = new Receiver(sender);
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(sender);
    exec.execute(receiver);
    TimeUnit.SECONDS.sleep(4);
    exec.shutdownNow();
  }
} /* Output: (65% match)
Read: A, Read: B, Read: C, Read: D, Read: E, Read: F, Read: G, Read: H, Read: I, Read: J, Read: K, Read: L, Read: M, java.lang.InterruptedException: sleep interrupted Sender sleep interrupted
java.io.InterruptedIOException Receiver read exception
*///:~
```



# 性能调优

辨认在java.util.concurrent类库当中哪些类适用于常规应用，哪些类只适用于提高性能

## 比较各类互斥技术

在互斥方法体很大时候，进入和退出互斥的开销可能很小，提高互斥速度可能对整体速度影响较小

- Lock
  - 在高并发下表现稳定
- synchronized
  - 高并发下非常不稳定
  - 在数量较小下表现更高效
  - 可读性好
- Atomic
  - 当一个对象的临界更新被限制为只涉及单个变量
  - 高并发下稳定

## 免锁容器

- 基于synchronized
  - vector
  - hashTable
- 免锁
  - 策略
    - 对容器的修改可以与读取操作同时发生，只要读取者只能看到完成修改的结果即可。
    - 修改时容器数据结构的某个部分的一个单独的副本，并且在修改过程中是不可视的，只有当修改完成时，被修改的结构才会自动与主数据结构交换
  - CopyOnWriteArrayList：整个数组复制
  - concurrentHashMap：部分内容可以复制和修改

乐观锁

- 只要你主要是从免锁容器中读取，就会比其synchronized对应物快很多。
- 如果需要向免锁容器中执行少量写入，情况依然如此

## 乐观加锁

CAS操作

## ReadWriteLock

- 对象数据结构相对不频繁地写入，但是有多个任务要经常读取的情况进行优化
- 如果写锁已经被其他任务持有，则如何读者都不能访问

# 活动对象

替换多线程模型的一种并发模型

- 对象是活动的，每个对象都维护着它自己的工作器线程和消息队列
- 有了活动对象，就可以串行化消息而不是方法。即不再需要防备一个任务在其循环的中间被中断
- 当向一个活动对象发送消息时，消息会被转换为一个任务，插入到足够对象的队列中，等待在以后的某个时刻运行

实现

java的***Future***

```Java
public class ActiveObjectDemo {
  private ExecutorService ex =
    Executors.newSingleThreadExecutor();
  private Random rand = new Random(47);
  // Insert a random delay to produce the effect
  // of a calculation time:
  private void pause(int factor) {
    try {
      TimeUnit.MILLISECONDS.sleep(
        100 + rand.nextInt(factor));
    } catch(InterruptedException e) {
      print("sleep() interrupted");
    }
  }
  public Future<Integer>
  calculateInt(final int x, final int y) {
    return ex.submit(new Callable<Integer>() {
      public Integer call() {
        print("starting " + x + " + " + y);
        pause(500);
        return x + y;
      }
    });
  }
  public Future<Float>
  calculateFloat(final float x, final float y) {
    return ex.submit(new Callable<Float>() {
      public Float call() {
        print("starting " + x + " + " + y);
        pause(2000);
        return x + y;
      }
    });
  }
  public void shutdown() { ex.shutdown(); }
  public static void main(String[] args) {
    ActiveObjectDemo d1 = new ActiveObjectDemo();
    // Prevents ConcurrentModificationException:
    List<Future<?>> results =
      new CopyOnWriteArrayList<Future<?>>();
    for(float f = 0.0f; f < 1.0f; f += 0.2f)
      results.add(d1.calculateFloat(f, f));
    for(int i = 0; i < 5; i++)
      results.add(d1.calculateInt(i, i));
    print("All asynch calls made");
    while(results.size() > 0) {
      for(Future<?> f : results)
        if(f.isDone()) {
          try {
            print(f.get());
          } catch(Exception e) {
            throw new RuntimeException(e);
          }
          results.remove(f);
        }
    }
    d1.shutdown();
  }
} /* Output: (85% match)
All asynch calls made
starting 0.0 + 0.0
starting 0.2 + 0.2
0.0
starting 0.4 + 0.4
0.4
starting 0.6 + 0.6
0.8
starting 0.8 + 0.8
1.2
starting 0 + 0
1.6
starting 1 + 1
0
starting 2 + 2
2
starting 3 + 3
4
starting 4 + 4
6
8
*///:~
```

活动对象的优势

- 每个对象都可以用于自己的工作区线程
- 每个对象都将维护对它自己的域的全部控制权
- 所有在活动对象间的通信都将以在这些对象间的消息形式发生
- 活动对象间的所有消息都要排队

# 参考 #

1. Java编程思想
