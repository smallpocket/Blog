---
title: Java接口：常用接口
type: tags
tags:
  - null
date: 2019-03-21 17:12:34
categories:
description:
---

# Cloneable接口

## 浅拷贝和深拷贝

浅拷贝是指拷贝对象时仅仅拷贝对象本身和对象中的基本变量，以及它所包含的所有对象的引用地址，而不拷贝对象包含的引用指向的对象（这是 java 中的浅拷贝，其他语言的浅拷贝貌似是直接拷贝引用，即新引用和旧引用指向同一对象）

深拷贝不仅拷贝对象本身和对象中的基本变量，而且拷贝对象包含的引用指向的所有对象

注意：对于常量池方式创建的 String 类型，会针对原值克隆，不存在引用地址一说，对于其他不可变类，如 LocalDate，也是一样

## Cloneable接口

使用clone克隆方法必须实现的接口

三句话总结：

（1）此类实现了Cloneable接口，以指示Object的clone()方法**可以合法地对该类实例进行按字段复制**

（2）如果在**没有实现Cloneable接口的实例上调用Object的clone()方法，则会导致抛出CloneNotSupporteddException**

（3）按照惯例，实现此接口的类应该**使用公共方法重写Object的clone()方法**，Object的clone()方法是一个受保护的方法

2、Object的clone()方法

创建并返回此对象的一个副本。对于任何对象x，表达式：

（1）**x.clone() != x为true**

（2）**x.clone().getClass() == x.getClass()为true**

（3）x.clone().equals(x)一般情况下为true，但这并不是必须要满足的要求

### 接口定义

Cloneable 没有定义任何的方法签名，clone 方法定义在 Object 类中：

```Java
protected native Object clone() throws CloneNotSupportedException;
```

从 JVM 的角度看，Cloneable 就是一个标记接口而已。到 clone() 的基本实现中，JVM 会去检测要 clone 的对象的类有没有被打上这个标记，有就让 clone，没有就抛异常。

Object 类的 clone() 一个 native 方法，native 方法的效率一般来说都远高于非 native 方法。这也解释了为什么要用 Object 中 clone() 方法而不是先 new 一个类，然后把原始对象中的信息赋到新对象中。

- 如果类没有实现 Cloneable 接口，直接调用从 Object 继承下来的 clone 方法，会抛出 CloneNotSupportedException

```Java
public class Test {  
    public static void main(String[] args) throws CloneNotSupportedException {  
        Test test = new Test();  
        Object cloned = test.clone();    // 结果是抛出 Clone NotSupportedException 异常 
    }  
}  
```

- 所有数组类型都有一个 public 的 clone 方法，而不是 protected。可以用这个方法建立一个新数组，包含原数组所有元素的副本。（数组类型由 JVM 独立实现）

```Java
int[] numbers = { 1, 2, 3, 4 };
int[] cloned = numbers.clone();
cloned[0] = 11;     // 不会改变 numbers
```

- Object 提供的 clone 方法是浅复制，如果要深克隆，需要重写（override）Object 类的 clone() 方法，并且在方法内部调用持有对象的 clone() 方法

```Java
public class Employee implements Cloneable {
   private String name;
   private double salary;
   private Date hireDay;

   public Employee clone() throws CloneNotSupportedException {
      // call Object.clone()
      Employee cloned = (Employee) super.clone();

      // clone mutable fields
      cloned.hireDay = (Date) hireDay.clone();

      return cloned;
   }
}
```

- 继承链上的祖先必须要有一个类声明实现 Cloneable 接口

```Java
public class Person {  
}  

public class Male extends Person implements Cloneable {  
    protected Object clone() throws CloneNotSupportedException {  
      return super.clone();  
    }  
}  

public class ChineseMale extends Male {  
}  

public class Test {  
    public static void main(String[] args) throws CloneNotSupportedException {  
        Person person = new Person();  
        Male male = new male();  
        ChineseMale chineseMale = new ChineseMale();  
        person.clone();          // 报错
        male.clone();  
        chineseMale.clone();
    }
}
```

## 为什么 clone() 是 protected 方法

clone() 方法是 protected 方法，为了让其它类能调用这个类的 clone() 方法，重载之后要把 clone() 方法的属性设置为 public 。

之所以把 Object 类中的 clone 方法定义为 protected，是因为若把 clone 方法定义为 public 时，就失去了安全机制。这样的 clone 方法会被子类继承，而不管它对于子类有没有意义。比如，我们已经为 Employee 类定义了 clone 方法，而其他人可能会去克隆它的子类 Manager 对象。Employee 克隆方法能完成这件事吗？这取决于 Manager 类中的字段类型。如果 Manager 的实例字段是基本类型，不会发生什么问题。但通常情况下，需要检查你所扩展的任何类的 clone 方法。

如果，只是进行浅度克隆，那就没有没有必要把它设写成 protected，public 就可以了。

总之，把 Object 类中的 clone 方法定义为 protected，就是确保深度克隆时派生类中的 clone 方法会被检查。

### protected 利与弊

一般不会使用 protected 域，因为会破坏类的封装性
 但是 protected 方法对于指示那些不提供一般用途而应在子类中重新定义的方法很有用

## 另一种克隆

事实上，若想实现对象的克隆，就不重写 Object 的 clone() 方法，还得实现 Cloneable 接口，有点麻烦。

而且如果某字段是个 **final** 的可变类，就不能将该引用指向原型对象新的克隆体了。

有更好的做法：

- 深克隆（deep clone/copy）： SerializationUtils
- 浅克隆（shallow clone/copy）：BeanUtils

简单的克隆，也可以通过编写拷贝构造方法实现

# 参考 #

1. [Cloneable 接口](https://www.jianshu.com/p/da8683e4d780)
