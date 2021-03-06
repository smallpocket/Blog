---
title: java并发
type: tags
tags:
  - 并发编程
date: 2019-02-18 23:05:11
categories: Java
description:
---

# 十一、线程安全

​	多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

## 概念

​	编写线程安全的代码，本质上就是管理对**状态**的访问，而且通常都是**共享、可变**的状态。

​	一个对象的状态就是它的数据，存储在状态变量中，如静态域、实例域。共享即一个变量可以被多个线程访问。可变即变量的值在其生命周期内都可以改变。真正要做到的线程安全是在不可控制的并发访问当中保护数据。

​	无论何时，只要有多于一个的线程访问给定的状态变量，而且其中某个线程会写入该变量，此时必须使用同步来协调线程对该变量的访问。

​	java的同步机制：synchronized(独占锁)，volatile(显示锁和原子变量的使用)

​	修复同步隐患

- 不跨线程的共享变量
- 使用状态变量为不可变的
- 在任何访问状态变量的时候使用同步

## 线程安全性

- 无状态的对象永远是安全的。即指这个对象没有状态域，也没有引用其他对象的域，是一次特定计算的瞬时状态，会唯一存放在一个本地变量当中，即线程的栈当中。而两个线程并不共享状态。

### 原子性

​	原子性：能作为一个单独的、不可分割的操作去执行

​	当向一个无状态的对象添加一个域，并进行long++操作（读+改+写），则不是线程安全

​	将long换作一个atomic包下的AtomicLong变量，则由于该变量是一个原子变量类，该计数器是线程安全的，该对象的状态即该计数器的对象，即该对象线程安全。（利用已有的线程安全类进行管理，如果只有一个，则线程安全，如果多个，则未必线程安全）

​	当变量之间相互关联，则在一个原子操作当中，要将几个相互关联的变量同时更新

**竞争条件**

​	想得到正确的答案，依赖时序。

### 锁

#### 内部锁（互斥锁）

​	synchronized块，声明方法

**特性**

- 重进入：内部锁是重进入的，当线程试图获得它自己所占有的锁时候，请求会成功，即重进入是基于每**线程**的，而不是调用。

  实现是通过为每个锁关联一个请求计数与一个占有它的线程，当同一线程访问，则计数++，线程退出该锁，则计数--，直到计数为0，释放该锁（父类与子类的使用）

**用锁来保持状态**

- 如果每个可被多个线程访问的可变状态变量，如果所有访问它的线程在执行状态当中占有同一个锁，则称该变量是由这个锁保护的
- 每个共享的可变变量都需要唯一一个确定的锁保护

**设计**

- 决定synchronize块大小需要权衡安全性（不能妥协）、简单性、性能。通常简单性与性能相互牵制，实现一个同步策略时候，不要过早地为了性能而牺牲简单性（是对安全性潜在的妥协）
- 有些耗时的计算或操作，如网络或者控制台IO，难以快速完成，执行它们的时候不要占有锁

### 活跃度与性能

​	不能武断地将整个方法设置为synchronize的，通过缩小synchronize的范围来提高并发性

![1550563393365](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550563393365.png)



## 共享对象

​	在前面，使用同步来避免多个线程在同一时间访问同一数据，在该部分，进行共享和发布对象，使得多个线程可以安全地访问他们。

​	同步的两个方面：

- 原子操作
- 内存可见性，即一个线程修改了对象的状态后，其他线程能够真正地看到改变

### 可见性

​	重排序现象：在单个线程当中，只要重排序不会对结果产生影响，则不能保证操作一定按照程序写定的顺序执行（java虚拟机的高性能）

​	如果数据需要跨线程共享，就要进行恰当的同步。

**过期数据**

​	当读线程检查一个变量时，可能看到一个过期的数据。并且过期不会发生在全部变量上，也不会完全不出现。

​	对于get与set方法，同样需要同步化，进行synchronize

**锁与可见性**

​	锁不仅仅是关于同步与互斥的，也是关于内存可见的，为了保证所有线程都能看到共享的、可变变量的最新值，读取和写入线程必须使用公共的锁进行同步

​	volatile：弱同步，确保对一个变量的更新以可预见的方式告知其他线程

### 发布和逸出

​	发布：发布一个对象指它能够被当前范围以外的代码所使用。发布一个对象，同时将发布该对象所有的非私有域引用的对象。

​	逸出：一个对象在尚未准备好的时候就发布

发布方式：

- 将对象存放到公共静态域

逸出：

- this引用在构造期间逸出，即对象在没有通过构造函数构造完毕（执行到了构造函数的某一句）时候逸出。

  当对象在构造函数当中创建一个线程，this引用总是被新线程共享，

### 线程封闭

​	访问共享的、可变的数据要求使用同步。一个可以避免同步的方法就是不共享数据，如果数据仅仅在单线程当中访问，则不需要任何同步。当对象封装在一个线程当中，则自动成为线程安全的。

- Swing将事件分发到线程当中
- JDBC从池中分配一个对象给线程。

**Ad-hoc线程限制**

​	指维护线程限制性的任务全部落在实现上的情况

- 确保只通过单一线程写入共享的volatile变量，则操作便是共享

**栈限制**

​	是线程限制的特例，只能通过本地变量才可以触及对象。本地变量使得对象更容易被限制在线程本地中**，本地变量本身就被限制在执行线程**中，它们存在于执行线程栈。其他线程无法访问这个栈

​	例如方法当中的numPairs

![1550587826303](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550587826303.png)

​	在该方法当中，实例化的animals只有一个引用指向它，因此它保存在线程的栈当中，倘若发布了animals或其内部对象的引用，则破坏了限制，并导致了对象逸出

**ThreadLocal**

​	更规范的方式，允许将每个线程与持有数值的对象关联在一起。ThradLocal提供了get和set，为每个使用它的线程维护一份单独的拷贝，所以get总是返回当前执行线程通过set设置的最新值。

> ThreadLocal 提供了线程本地的实例。它与普通变量的区别在于，每个使用该变量的线程都会初始化一个完全独立的实例副本。ThreadLocal 变量通常被`private static`修饰。当一个线程结束时，它所使用的所有 ThreadLocal 相对的实例副本都可被回收。

​	总的来说，**ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用，也即变量在线程间隔离而在方法或类间共享的场景**

### 不可变性

​	创建后不能被修改的对象称为不可变对象，不可变对象永远是线程安全的。

不可变对象特征：

- 状态创建后不能再修改
- 所有域都是final类型（不获得可变对象的引用）
- 对象被正确创建，不逸出this

### 安全发布

​	如果希望跨线程共享对象，则必须安全地共享它

​	对象的引用对其他线程可见，但它的状态可能是过期的，即对象的状态不一定对消费线程可见。

**安全发布的模式**

- 通过静态初始化器初始化对象的引用（JVM内部的同步机制）
- 将它的引用存储到volatile域或者atomicReference
- 将它的引用存储到正确创建的对象的final域
- 或者将它的引用存储到由锁正确保护的域中

**线程安全容器**

​	线程安全容器的内部同步，即将对象置入这些容器的操作符合最后一条要求

- HashTable、synchronizedMap、concurrentMap
- Vector、CopyOnWriteArrayList、synchronizedList
- BlockingQueue、concurrentLinkedQueue

**高效不可变对象**

​	一个对象在技术上不是不可变得，但是它的状态在发布后不会再更改，即有效不可变对象。

> 任何线程都可以在没有额外同步的情况下安全使用一个**安全发布**的高效不可变对象

**可变对象**

​	安全发布仅仅保证发布当时的可见性，对于可变性，还需要线程安全或锁

- 可变对象必须要安全发布，同时必须要线程安全或者是锁保护的

  > - 线程限制：一个线程限制的对象，通过限制在线程中，而被线程独占，且只能被占有它的线程修改
  > - 共享只读：在没有额外同步的情况下可以被多个对象并发访问，但是任何线程都不可以修改它，包括可变对象和高效不可变对象
  > - 共享线程安全：一个线程安全的对象在内部同步，所以其他线程无须额外同步，就可以通过公共接口访问
  > - 被守护的：一个被守护的对象只能通过特定的锁来访问。被守护的对象包括那些被线程安全对象封装的对象，和已知被特定的锁保护起来的已发布对象

# 组合对象

​	构造类的模式，使得类更容易成为线程安全的，并且不会使程序意外破坏类的线程安全性

## 设计线程安全的类



> 设计线程安全类的过程的基本要素
>
> - 确定对象状态是由哪些变量构成的（linkedList状态包含了所有存储在链表中节点对象的状态）
> - 确定限制状态变量的不变约束
> - 制定一个管理并发访问对象状态的策略

### 同步策略

> 同步策略定义了对象如何协调对其状态的访问，并且不会违反它的不变约束或后验条件。规定了如何把不可变性、线程限制、锁结合起来，从而维护线程的安全性，还指明了哪些锁维护哪些变量

​	状态空间：对象与变量可能处于状态的范围

​	**操作的后验条件**：对状态的值进行检验，如果不符合，则异常

​	状态转换：一个对象的下一个状态源于当前状态。如果某些状态是非法的，则必须封装该状态下的状态变量，否则客户代码会将对象置于非法状态。如果一个操作的过程当中出现非法状态，则该操作必须是原子的

**状态依赖的操作**

​	某些对象的方法基于对象的先验条件，例如无法从空队列移除一个对象。在多线程当中，原本为假地先验条件可能在后续为真

**状态所有权**

## 实例限制

​	即使对象不是线程安全的，依然可以使得它安全用于多线程，比如确保其只在单一线程被访问（线程限制）、类实例（private的类成员）、语汇范围（本地变量），要确保不逸出

> ​	将数据封装在对象内部，把对数据的访问限制在对象的方法（线程安全的锁或。。。）上，更容易确保线程在访问数据时总能获得正确的锁。
>
> ​	限制性使得构造线程安全的类变得容易，因为类的状态被限制后，分析它的线程安全性便不需要检查完整的程序。

**java监视器模式**

​	遵循java监视器模式的对象封装了所有的可变状态，并且由对象自己的内部锁保护

​	对于凭空创建一个类，或者使用非线程安全的对象组装一个类时，非常有效

![1550634538699](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550634538699.png)

​	内部锁：将synchronize添加到方法上

## 委托线程安全

​	如果类的状态组件都是线程安全的，则监视器模式的使用需要视情况而定，它是一个额外的线程安全层。如果类组件都是线程安全的，那么可以说类将它的线程安全委托给了它的组件，因为组件安全，所以它也是安全的。

​	如果将线程安全委托到多个隐含的状态变量上，只要这些变量相互独立，组合对象未增加任何涉及多个状态变量的**不变约束**

> ​	如果一个类由多个彼此相互独立的线程安全的状态变量组成，并且类的操作不包含任何无效状态转换时，可以将线程安全委托给这些状态变量

**发布底层的状态变量**（类组件）

> ​	如果一个状态变量是线程安全的，没有任何的不变约束限制它的值，并且没有任何状态转换限制它的操作，那么它可以被安全发布

## 向已有的线程安全类添加功能

- 修改原始的类。但通常不可能，首先需要理解原始类的同步策略，其次要有修改源码的权限
- 继承扩展，但并非所有类给子类暴露了足够多的状态，并且子类可能被底层类不知不觉破坏
- 扩展功能。

**客户端加锁**

​	将扩展代码与类置于一个新类当中

![1550636338076](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550636338076.png)

​	但是这样并不正确，因为给方法加锁，但是这个锁与list的锁并不一致，方法所用的锁与list用于客户端加锁与外部加锁时用的锁并不一致。

​	正确写法：

![1550641893865](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550641893865.png)

​	相比于扩展类，在客户端加锁更加脆弱，会破坏同步策略的封装性

**组合**

​	使用内部锁，基于java监视器实现了新的一层锁，只要持有list的唯一引用，则是线程安全的

![1550642087788](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1550642087788.png)

# 构建块

​	线程安全容器以及多种同步工具（synchronizer），synchronizer可以调节相互协作的线程间的控制流。

## 同步容器

​	包括Vector和HashTable，这些类通过封装它们的状态，并对每一个公共方法进行同步实现了线程安全，这样一次只能有一个线程访问容器。通过对容器的所有状态串行访问实现的线程安全，削弱了并发性。

​	容器本身是线程安全的，无论有多少线程同时调用容器，也不会破坏容器。但是对于方法的调用者来说，当线程在并发地修改容器，最后得到的结果并不是所预期的结果。面对这种情况，需要对容器类自身进行加锁，synchronized(list)。保证复合操作的原子性

**迭代**

​	对容器进行迭代的时候，需要对容器进行加锁，防止容器数据更改，但是这么一来对并发的性能就大大下降，会在相当长时间加锁，甚至产生死锁。一个解决办法是复制容器，因为是存在线程当中，但是在复制过程中依然需要加锁，而且占用空间。

> ​	正如封装一个对象的状态，能够使它更容易地保持不变约束一样，封装它的同步则可以迫使它符合同步策略

​	隐藏迭代器：有些迭代器是隐藏的，比如toString方法，hashCode方法，equals方法都会对容器进行迭代

## 并发容器

​	为多线程的并发访问而设计。concurrentHashMap代替同步的HashMap，当多数操作为读取，CoayOnWriteArrayList是List的同步，concurrentMap接口增加了常见复合操作的支持

> ​	用并发容器替换同步容器，这种做法以有很小的风险带来了可扩展性显著提高

**ConcurrentHashMap**

​	在该容器前，程序使用一个公共锁同步每一个方法，明确只有一个线程可以访问容器

​	ConcurrentHashMap使用更细化的锁机制，即**分离锁**，任意数量的读线程可以并发访问，读者与写者可以并发访问，有限数量的写进程可以并发修改。该容器的迭代器具有**弱一致性**，即该迭代器容许并发修改，它会感应到迭代器被创建后对容器的修改。

​	在独占的访问加锁中，该容器无法胜任

## 阻塞和可中断方法

​	阻塞状态：blocked，waiting，Timed_waiting

​	中断：thread.interrupt()，中断是一种协作机制，

## synchronizer



# 构建并发应用程序

​	大多数的并发程序是围绕**任务**进行管理，任务即抽象、离散的工作单元

## 任务执行

**在线程中执行任务**

指明任务的边界，使得任务为一个独立的活动，不依赖其他任务的状态、结果、边界效应。

任务边界：

- 单独的客户请求

任务执行策略：

- 顺序执行

  即收到一个请求，服务器要处理完该请求才能继续accept下一个请求

![1550825942991](assets\1550825942991.png)

- 显式地为任务创建线程，为每一个服务请求创建一个线程
  - 并行处理，并且能同时接受多个请求
  - 任务处理代码必须线程安全
  - 线程生命周期的开销、资源消耗、无限制线程的稳定性拖垮系统

![1550825994345](assets\1550825994345.png)

**Exeutor框架**

​	基于生产者-消费者模式，提交任务的执行者是生产者，执行任务的线程是消费者。

​	该框架可以用于异步任务执行，并且支持多种类型的任务执行策略，为任务提交与任务执行做了解耦。

![1550988548365](assets\1550988548365.png)

**执行策略**

​	任务的提交与任务的执行解耦，可以更简单地为一个类给定的任务制定执行策略，一个执行策略包含“what where when how”的因素

- 任务在什么线程执行 what
- 任务以什么顺序执行 what (FIFO,LIFO,优先级)
- 可以由多少个任务并发执行 how many
- 可以有多少个任务进入等待执行队列 how many
- 如果系统过载，需要放弃一个任务，选择哪一个任务？如何通知应用程序 which
- 在一个任务执行前与后，应该做什么处理 what

**生命周期**

​	由于exectuor异步执行任务，所以之前提交的任务状态不能立即可见（有些已经完成，有些正在运行，有些在队列当中），如果将其关闭，可能出现各种问题

## 取消和关闭

​	中断，一种协作机制，使得一个线程能够要求另一个线程停止当前工作

**任务取消**

​	原因

- 用户请求的取消，cancel
- 限时活动，超时取消
- 应用程序事件，当程序的不同任务在搜索，一个任务找到了解决方案，其他任务就取消
- 错误，IO等错误
- 关闭，

## 线程池

​	线程池与工作队列（持有所有等待执行的任务）绑定，线程从工作队列获取下一个任务，执行。

优势

- 重用存在的线程，抵消线程创建、消亡的开销
- 在请求到达时，线程已存在，创建线程的等待时间不会延误任务的执行
- 可以防止创建过多的线程，争夺资源

死锁

性能与可伸缩性

显式锁

# 参考 #

1. Java并发编程实战
