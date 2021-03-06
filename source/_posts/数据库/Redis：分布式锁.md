---
title: Redis：分布式锁
type: tags
tags:
  - Redis
  - NoSQL
date: 2019-04-16 23:18:37
categories: 数据库
description:
---

# Redis分布式锁简单实现

熟悉Redis的同学那么肯定对setNx(set if not exist)方法不陌生，如果不存在则更新，其可以很好的用来实现我们的分布式锁。对于某个资源加锁我们只需要

```SQL
setNx resourceName value
```

这里有个问题，加锁了之后如果机器宕机那么这个锁就不会得到释放所以会加入过期时间，加入过期时间需要和setNx同一个原子操作，在Redis2.8之前我们需要使用Lua脚本达到我们的目的，但是redis2.8之后redis支持nx和ex操作是同一原子操作。

```SQL
set resourceName value ex 5 nx
```

# Redission

Javaer都知道Jedis，Jedis是Redis的Java实现的客户端，其API提供了比较全面的Redis命令的支持。Redission也是Redis的客户端，相比于Jedis功能简单。Jedis简单使用阻塞的I/O和redis交互，Redission通过Netty支持非阻塞I/O。Jedis最新版本2.9.0是2016年的快3年了没有更新，而Redission最新版本是2018.10月更新。

Redission封装了锁的实现，其继承了java.util.concurrent.locks.Lock的接口，让我们像操作我们的本地Lock一样去操作Redission的Lock，下面介绍一下其如何实现分布式锁。

![img](assets/16652989862ce5af)

Redission不仅提供了Java自带的一些方法(lock,tryLock)，还提供了异步加锁，对于异步编程更加方便。 由于内部源码较多，就不贴源码了，这里用文字叙述来分析他是如何加锁的，这里分析一下tryLock方法:

1. 尝试加锁:首先会尝试进行加锁，由于保证操作是原子性，那么就只能使用lua脚本，相关的lua脚本如下： 

   ![img](assets/166529eb8139751a)可以看见他并没有使用我们的sexNx来进行操作，而是使用的hash结构，我们的每一个需要锁定的资源都可以看做是一个HashMap，锁定资源的节点信息是Key,锁定次数是value。通过这种方式可以很好的实现可重入的效果，只需要对value进行加1操作，就能进行可重入锁。当然这里也可以用之前我们说的本地计数进行优化。

2. 如果尝试加锁失败，判断是否超时，如果超时则返回false。

3. 如果加锁失败之后，没有超时，那么需要在名字为redisson_lock__channel+lockName的channel上进行订阅，用于订阅解锁消息，然后一直阻塞直到超时，或者有解锁消息。

4. 重试步骤1，2，3，直到最后获取到锁，或者某一步获取锁超时。

对于我们的unlock方法比较简单也是通过lua脚本进行解锁，如果是可重入锁，只是减1。如果是非加锁线程解锁，那么解锁失败。 

![img](assets/16652acd8c664482)



Redission还有公平锁的实现，对于公平锁其利用了list结构和hashset结构分别用来保存我们排队的节点，和我们节点的过期时间，用这两个数据结构帮助我们实现公平锁，这里就不展开介绍了，有兴趣可以参考源码。

## 底层原理

![img](assets/16730ecd592f41e3)

### 加锁机制

现在某个客户端要加锁。如果该客户端面对的是一个redis cluster集群，他首先会根据hash节点选择一台机器。**这里注意**，仅仅只是选择一台机器！这点很关键！紧接着，就会发送一段lua脚本到redis上，那段lua脚本如下所示：

![img](assets/166529eb8139751a)

LUA脚本：保证复杂业务逻辑执行的**原子性**。

- **KEYS[1]**代表的是你加锁的那个key，比如说：RLock lock = redisson.getLock("myLock");这里你自己设置了加锁的那个锁key就是“myLock”。

- **ARGV[1]**代表的就是锁key的默认生存时间，默认30秒。**ARGV[2]**代表的是加锁的客户端的ID，类似于下面这样：8743c9c0-0795-4907-87fd-6c719a6b4586:1

- 第一段if判断语句，就是用“**exists myLock**”命令判断一下，如果你要加锁的那个锁key不存在的话，你就进行加锁。如何加锁呢？很简单，**用下面的命令**：hset myLock 8743c9c0-0795-4907-87fd-6c719a6b4586:1 1，通过这个命令设置一个hash数据结构，这行命令执行后，会出现一个类似下面的数据结构：

  ```
  myLock:{
      "8743c9c0-0795-4907-87fd-6c719a6b4586:1" : 1;
  }
  ```

- 代表“8743c9c0-0795-4907-87fd-6c719a6b4586:1”这个客户端对“myLock”这个锁key完成了加锁。接着会执行“**pexpire myLock 30000**”命令，设置myLock这个锁key的**生存时间是30秒**。好了，到此为止，ok，加锁完成了。


### 释放锁机制

如果执行lock.unlock()，就可以释放分布式锁，此时的业务逻辑也是非常简单的。其实说白了，就是每次都对myLock数据结构中的那个加锁次数减1。如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：**“del myLock”命令**，从redis里删除这个key。然后呢，另外的客户端2就可以尝试完成加锁了。这就是所谓的**分布式锁的开源Redisson框架的实现机制。**

一般我们在生产系统中，可以用Redisson框架提供的这个类库来基于redis进行分布式锁的加锁与释放锁。

### 锁互斥机制

那么在这个时候，如果客户端2来尝试加锁，执行了同样的一段lua脚本，会咋样呢？很简单，第一个if判断会执行“**exists myLock**”，发现myLock这个锁key已经存在了。接着第二个if判断，判断一下，myLock锁key的hash数据结构中，是否包含客户端2的ID，但是明显不是的，因为那里包含的是客户端1的ID。

所以，客户端2会获取到**pttl myLock**返回的一个数字，这个数字代表了myLock这个锁key的**剩余生存时间。**比如还剩15000毫秒的生存时间。此时客户端2会进入一个while循环，不停的尝试加锁。



### watch dog自动延期机制

客户端1加锁的锁key默认生存时间才30秒，如果超过了30秒，客户端1还想一直持有这把锁，怎么办呢？

简单！只要客户端1一旦加锁成功，就会启动一个watch dog看门狗，**他是一个后台线程，会每隔10秒检查一下**，如果客户端1还持有锁key，那么就会不断的延长锁key的生存时间。



### 可重入加锁机制

那如果客户端1都已经持有了这把锁了，结果可重入的加锁会怎么样呢？

这时我们来分析一下上面那段lua脚本。**第一个if判断肯定不成立**，“exists myLock”会显示锁key已经存在了。**第二个if判断会成立**，因为myLock的hash数据结构中包含的那个ID，就是客户端1的那个ID，也就是“8743c9c0-0795-4907-87fd-6c719a6b4586:1”

此时就会执行可重入加锁的逻辑，他会用：

incrby myLock 8743c9c0-0795-4907-87fd-6c71a6b4586:1 1  ，通过这个命令，对客户端1的加锁次数，累加1。此时myLock数据结构变为下面这样：

```C
myLock:{
    "8743c9c0-0795-4907-87fd-6c719a6b4586:1" : 2
}
```

大家看到了吧，那个myLock的hash数据结构中的那个客户端ID，就对应着加锁的次数



### 释放锁机制

如果执行lock.unlock()，就可以释放分布式锁，此时的业务逻辑也是非常简单的。其实说白了，就是每次都对myLock数据结构中的那个加锁次数减1。如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：**“del myLock”命令**，从redis里删除这个key。然后呢，另外的客户端2就可以尝试完成加锁了。这就是所谓的**分布式锁的开源Redisson框架的实现机制。**

一般我们在生产系统中，可以用Redisson框架提供的这个类库来基于redis进行分布式锁的加锁与释放锁。

### 上述Redis分布式锁的缺点

其实上面那种方案最大的问题，就是如果你对某个redis master实例，写入了myLock这种锁key的value，此时会异步复制给对应的master slave实例。但是这个过程中一旦发生redis master宕机，主备切换，redis slave变为了redis master。

接着就会导致，客户端2来尝试加锁的时候，在新的redis master上完成了加锁，而客户端1也以为自己成功加了锁。此时就会导致多个客户端对一个分布式锁完成了加锁。这时系统在业务语义上一定会出现问题，**导致各种脏数据的产生**。

所以这个就是redis cluster，或者是redis master-slave架构的主从异步复制导致的redis分布式锁的最大缺陷：**在redis master实例宕机的时候，可能导致多个客户端同时完成加锁**。



# RedLock

我们想象一个这样的场景当机器A申请到一把锁之后，如果Redis主宕机了，这个时候从机并没有同步到这一把锁，那么机器B再次申请的时候就会再次申请到这把锁，为了解决这个问题Redis作者提出了RedLock红锁的算法,在Redission中也对RedLock进行了实现。



![img](assets/16652bd95e11a8b3)



通过上面的代码，我们需要实现多个Redis集群，然后进行红锁的加锁，解锁。具体的步骤如下:

1. 首先生成多个Redis集群的Rlock，并将其构造成RedLock。
2. 依次循环对三个集群进行加锁，加锁的过程和5.2里面一致。
3. 如果循环加锁的过程中加锁失败，那么需要判断加锁失败的次数是否超出了最大值，这里的最大值是根据集群的个数，比如三个那么只允许失败一个，五个的话只允许失败两个，要保证多数成功。
4. 加锁的过程中需要判断是否加锁超时，有可能我们设置加锁只能用3ms，第一个集群加锁已经消耗了3ms了。那么也算加锁失败。
5. 3，4步里面加锁失败的话，那么就会进行解锁操作，解锁会对所有的集群在请求一次解锁。

可以看见RedLock基本原理是利用多个Redis集群，用多数的集群加锁成功，减少Redis某个集群出故障，造成分布式锁出现问题的概率。

# Redis小结

- 优点:对于Redis实现简单，性能对比ZK和Mysql较好。如果不需要特别复杂的要求，那么自己就可以利用setNx进行实现，如果自己需要复杂的需求的话那么可以利用或者借鉴Redission。对于一些要求比较严格的场景来说的话可以使用RedLock。
- 缺点:需要维护Redis集群，如果要实现RedLock那么需要维护更多的集群。

# 参考 #

1. [再有人问你分布式锁，这篇文章扔给他](<https://juejin.im/post/5bbb0d8df265da0abd3533a5#heading-18>)
