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

# 不可变对象

有一种对象只要发布了就是安全的，即不可变对象。例如string类

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



# 参考 #

1. 
