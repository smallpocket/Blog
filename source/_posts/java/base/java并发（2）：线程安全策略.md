---
title: java并发（2）：线程安全策略
type: tags
tags:
  - null
date: 2019-03-01 15:42:55
categories:
description:
---

# 线程安全策略

- 线程限制
  - 一个被线程限制的对象，由线程独占，并且只能被占有它的线程修改
  - 线程封闭：把对象封装到一个线程当中，只有一个线程可以看到它
- 共享只读
  - 一个共享只读的对象，在没有额外同步的情况下，可以被多个线程并发访问，但是任何线程都不能修改它
  - 不可变对象：一种对象只要发布了就是安全的，即不可变对象，是一种躲避并发的方法

- 线程安全对象
  - 一个线程安全的对象或者容器，在内部通过同步机制来保证线程安全，所有其他线程无需额外同步就可以通过公共接口随意访问它
- 被守护对象
  - 只能通过获取特定的锁来访问

# 不可变对象

有一种对象只要发布了就是安全的，即不可变对象，是一种躲避并发的方法。例如string类。

不可变对象需要满足的条件

- 对象创建以后状态就不能修改
  - 将类声明为final
- 对象所有域都是final类型
  - 所有域声明为私有
  - 不通过set方法
  - 将所有可变数据声明为final
- 对象是正确创建的，this引用没有逸出
  - 通过构造器初始化所有成员
  - 在get方法不直接返回对象本身，而是返回一个clone

**final关键字**：类、方法、变量

- 修饰类：
  - 不能被继承
  - 所有成员方法会隐式选择为final
- 修饰方法
  - 锁定方法不能被继承修改
- 修饰变量
  - 基本数据类型变量
  - 引用类型变量（初始化后，不能指向另一个对象）

其他创建不可变对象方法

- Collections.unmodifiableXXX：Collection、List、Set、Map...

```java
public class ImmutableExample2 {

    private static Map<Integer,Integer> map = Maps.newHashMap();

    static {
        map.put(1,2);
        map.put(3,4);
        map.put(5,6);
        //创建final的map
        map = Collections.unmodifiableMap(map);
    }

    public static void main(String[] args) {
        //会抛出异常, map无法被修改
        map.put(1,3);
        log.info("{}",map.get(1));
    }
}
```

将返回一个新的map，将数据拷贝过去，然后将所有更改数据转换为了抛出异常

```Java
    public static <K,V> Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m) {
        return new UnmodifiableMap<>(m);
    }
```

- Guava：ImmutableXXX：Collection、List、Set、Map...

```Java
public class ImmutableExample3 {

    private final static ImmutableList<Integer> list = ImmutableList.of(1,2,3);

    private final static ImmutableSet set = ImmutableSet.copyOf(list);

    private final static ImmutableMap<Integer,Integer> map = ImmutableMap.of(1,2,3,4);

    private final static ImmutableMap<Integer,Integer> map2 = ImmutableMap.<Integer,Integer>builder()
            .put(1,2).put(3,4).put(5,6).build();

    public static void main(String[] args) {
//        set.add(4);
//        map2.put(1,4);
        System.out.println(map2.get(3));
    }

}
```

# 线程封闭

把对象封装到一个线程当中，只有一个线程可以看到它。

**线程封闭方法：**

- Ad-hoc线程封闭：程序控制实现，最糟糕。
- 堆栈封闭：局部变量，无并发问题。
- ThreadLocal线程封闭：特别好的封闭方法。
  - 内部维护了一个map，key是线程名称，值是对象

# 线程不安全类与写法

线程不安全类：

- 如果一个类的对象可以同时被多个线程访问，如果没有做并发处理，则会出现异常

StringBuilder是线程不安全的。

StringBuffer是线程安全的，它内部方法添加了synchronized，但也因此它的性能有损耗。

**ArrayList、hashMap、hashSet等collection**

线程不安全

# 同步容器

类别

- ArrayList->Vector、Stack
- HashMap->HashTable
- Collections.synchronizedXXX(List、Set、Map)
  - collection的静态工厂创建

同步容器也未必线程安全

同步容器虽然保证了同一时刻只有一个线程可以访问，但是线程交替进行访问依然会出现问题

```Java
                Thread thread1 = new Thread() {
                    public void run() {
                        for (int i = 0; i < vector.size(); i++) {
                            vector.remove(i);
                        }
                    }

                };

                Thread thread2 = new Thread() {
                    public void run() 
                    //在size=10,i=9时刻,上面的线程将其删除,而此时读取则会出现异常
                        for (int i = 0; i < vector.size(); i++) {
                            vector.get(i);
                        }
                    }
                };

                thread1.start();
                thread2.start();
```

# 并发容器J.U.C

JDK下的一个包Java.util.concurrent

- ArrayList->CopyOnWriteArrayList
  - 写操作的时候复制，在新的数组上操作，写完后，将原有的数据指向新数组
  - 拷贝数组消耗内存
  - 不能用于实时读数组，可能读取到旧的，适合读多写少
- HashSet、TreeSet->CopyOnWriteArraySet、ConcurrentSkipListSet
  - ConcurrentSkipListSet：支持自然排序（TreeSet），基于map。对于批量操作并不能保证原子操作，对于单个cotains操作可以
- HashMap、TreeMap->ConcurrentHashMap、ConcurrentSkipListMap
  - ConcurrentHashMap:针对读操作做了大量的优化，能应付很大的并发，速度快
  - ConcurrentSkipListMap：TreeMap的安全版，基于跳表实现，支持更高的并发，线程越多越优秀
- Collections.synchronizedXXX(List、Set、Map)

![1551612197603](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551612197603.png)

## HashMap与ConcurrentHashMap

HashMap

数据结构：

- 数组
- 引用

![1551689310736](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551689310736.png)

参数

- 初始容量

- 加载因子

寻址方式

- 取模操作消耗操作较大，使用与操作与2^n-1进行与运算

## 同步器AQS

# 参考 #

1. 
