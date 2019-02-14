---
title: 泛型
type: tags
tags:
  - javaBase
date: 2018-10-17 20:46:31
categories: java
description: 泛型的使用
---

# 基本概念

## 作用

- 保证类的可重用性
- 指定容器要持有什么类型的对象，并由编译器保证类型的正确性

# 应用

## 泛型类

使用泛型编写一个更通用的类

当处理一个问题，这个问题需要解决一个对应类，便需要在类当中进行组合。但如果是一组类，我们可能就需要些一组类，无疑是很麻烦的一件事情。这种时候，可以使用泛型

### 个例问题

```java
package fifteen.generics;

import eleven.collection.Pet;

/**
 * 编写一个通用类的尝试
 *
 * @Author : Heper
 * @Time : 2019/1/28 19:05
 */
public class Holder {

    /**
     * 持有一个Object的类,对它进行处理
     * 但是,如此一来,并不能持有一个其他的类,必须要进行向上转型
     * 如果说,想要持有一个新的类,就又要写一个新的类
     * 使用泛型可以帮助解决这种问题
     */
    private Object a;


    public Holder(Object a) {
        this.a = a;
    }

    public void set(Object a) {
        this.a = a;
    }

    public Object get() {
        return a;
    }

    public static void main(String[] args) {
        Holder h2 = new Holder(new Pet());
        Pet a = (Pet) h2.get();
        h2.set("Not an Automobile");
        String s = (String) h2.get();
        h2.set(1);
        Integer x = (Integer) h2.get();
    }
}

```

### 通用的解决方案

```java
package fifteen.generics;

import eleven.collection.Pet;

/**
 * 基于泛型对类的复用性进行扩展
 *
 * @Author : Heper
 * @Time : 2019/1/28 19:59
 */
//public class Holder3<T, E> {
public class Holder2<T> {
    /**
     * 可以看到,这个类从Object变成了T
     * 即一个可以在编译时更改的类,则在创建对象时候,只需要指明类型即可
     */
    private T a;

    public Holder2(T a) {
        this.a = a;
    }

    public void set(T a) {
        this.a = a;
    }

    public T get() {
        return a;
    }

    public static void main(String[] args) {
        //创建对象时候,要指明T的类型
        Holder2<Pet> h3 =
                new Holder2<>(new Pet());
        Pet a = h3.get(); // No cast needed
        // h3.set("Not an Automobile"); // Error
        // h3.set(1); // Error
    }
}

```

## 泛型接口

## 泛型方法

使用泛型方法需要在返回类型前加<T>

```java
/**
 * 泛型方法
 *
 * @Author : Heper
 * @Time : 2019/2/2 17:10
 */
public class GenericMethods {
    public <T> void f(T x) {
        System.out.println(x.getClass().getName());
    }

    public static void main(String[] args) {
        /**
         * 对f方法的重载
         * 编译器自动根据参数的类型进行重载
         */
        GenericMethods gm = new GenericMethods();
        gm.f("");
        gm.f(1);
        gm.f(1.0);
        gm.f(1.0F);
        gm.f('c');
        gm.f(gm);
    }
}
```



# 参考 #

1. 
