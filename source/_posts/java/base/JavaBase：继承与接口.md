---
title: javaBaseInherit
type: tags
date: 2018-09-23 10:30:10
categories: java
tags: 
- JavaBase
description: java的继承、组合以及优缺点，以及抽象类、如何设计参数数量可变的方法，以及一点点的反射
---
# 继承 #

- java虚拟机对待继承时,会在类先寻找该方法,如果找不到,就找它的父类,依次寻找上级直到找到为止
- 继承关系可以通过is-a关系检验
- public成员会被继承,private成员不会被继承 
  
## 子类与父类的联系 ##

- 子类不能访问父类的私有变量,但是!!!子类在创建的时候,还是有初始化父类的私有变量的  
- 当初始化的时候,子类会创建子类,父类以及object类的对象,并会把父类和object类包含在内部,(还是一个对象)

## 阻止继承：final类与方法

- 使用final可以使得类无法被继承，同时，也会使得方法无法被子类覆盖
- final类中的所有的方法自动成为final方法
- final类中的域不会自动final

### 举例

String类就是一个final类，如果存在一个String的引用，它引用的一定是一个String，而不可能是其他的，因为它没有子类

### 为什么要使用

确保不会在子类中改变语义

## proteced受保护访问

希望超类中的某些方法允许被子类访问，或允许子类的方法访问超类的某个域，则将其定义为proteced，则子类可以直接访问

## 注意 ###

- C#中(Java应该一样)虽然private成员不会被继承,但是可以继承public方法,以此去访问成员,   
- 但是子类是没有那个private成员的!!!!,它没法继承那个成员,它所得到的只是通过继承父类的public方法,以该方法去访问那个成员

# Object所有类的超类

所有类的超类都是object，每个类都是由它扩展而来的

## equals方法

- 检验一个对象是否等于另一个对象
- 而原生的方法判断的是对象的两个引用是否引用的是同一个对象。所以一般较没有意义
- 而经常需要的equals方法是判断其属性等方法是否一致，因此这些时候需要对equals进行重写

## hashCode方法

- hashCode是对象的存储地址，如果两个对象不相同，那么他们的code也不会相同
- 字符串的code是根据内容导出的


# 关于继承和组合 #

- 组合：使用现有的类合成一个新的类
- 聚合：动态的进行组合

## 如何选择继承和组合 ## 

1. 组合相对于继承更为灵活，过分地使用继承会导致设计过分地复杂，而且并不清晰
2. 建议在同样可行的情况下，优先使用组合而不是继承。因为组合更安全，更简单，更灵活，更高效。
3. 问一问自己是否需要从新类向基类进行向上转型。如果是必须的，则继承是必要的。反之则应该好好考虑是否需要继承。
4. 只有当子类真正是超类的子类型时，才适合用继承。换句话说，对于两个类A和B，只有当两者之间确实存在is-a关系的时候，类B才应该继续类A。

## 优缺点 ##

### 组合优点：###

1. 不破坏封装，整体类与局部类之间松耦合，彼此相对独立
2. 具有较好的可扩展性
3. 支持动态组合。在运行时，整体对象可以选择不同类型的局部对象
4. 整体类可以对局部类进行包装，封装局部类的接口，提供新的接口

### 组合缺点：### 

1. 整体类不能自动获得和局部类同样的接口
2. 创建整体类的对象时，需要创建所有局部类的对象

### 继承优点：### 

1. 子类能自动继承父类的接口
2. 创建子类的对象时，无须创建父类的对象

### 继承缺点：### 

1. 破坏封装，子类与父类之间紧密耦合，子类依赖于父类的实现，子类缺乏独立性
2. 支持扩展，但是往往以增加系统结构的复杂度为代价
3. 不支持动态继承。在运行时，子类无法选择不同的父类

# 抽象类

abstract类

抽象类可以包含具体的数据与方法，抽象方法起到一个占位的作用

# 对象包装器与自动装箱

将int等基本数据类型转换为对象，有Integer类等

对象包装器类是不可变的类，一旦构造了之后，其值便无法改变，并且，对象包装器是final类

## 自动装箱与拆箱

自动装箱：list.add(3)将转变为list.add(Integer.valueOf(3))

自动拆箱：int a =list.get(i)，将自动转换为int a=list.get(i).intValue()

# 参数数量可变的方法

使用...

```java
/**
 *在这里面，value是一个数组，可以传入多个参数
 * 可以调用max(-1,2,3)
 */
public double max(double... value){
    
}
```


# 反射

## 反射定义

能够分析类能力的程序称为反射

### 反射的作用

- 在运行时分析类的能力
- 在运行时查看对象，例如，编写一个toString供所有方法使用
- 实现通用的数组操作代码
- 利用method对象

## Class类

java运行时系统始终为所有的对象维护一个被称为运行时的类型标识

# 接口

接口用于描述类的功能

## 接口的实现

- 使用 Interface关键字
- 可以定义常量（public static final），但是不能定义实例域
- 在SE8之后可以定义方法，但是不能使用实例域，因为没有实例

### 默认方法

为接口的方法提供一个默认的实现

```java
/**
 * 所有继承了该接口的方法，如果没有重写该方法，则会使用默认实现
 * 可以将全部方法声明为默认实现，则就只需要去重写真正需要的方法
 */
default int compareTo(T other){
	return 0;
}
```

实现“接口演化”

- 如果为一个接口新增了一个方法，那么所有实现该接口的类都要去覆盖这个方法，就会导致旧版代码的不兼容性。
- 使用默认实现，则以前的类并不需要去覆盖该方法，只有有需求的类才会去覆盖

## 实现了接口就履行了合约

1.履行合约的方式是去定义相关的方法  
2.履行了合约,即可去实现与合约相关的动作,实现类似继承和多态

# 对象克隆Cloneable接口

克隆一个对象并不能简单的让一个引用去等于另一个引用，因为它们将指向同一个对象。

若想让它们在初始状态相同，而在之后会发生独立的改变，则需要对对象进行克隆。

若想对对象进行克隆，则需要使用clone方法，clone方法是Object的一个protected方法，因此，假设有Employee类，则只有Employee类才能clone Employee类。

## 克隆的困难

- 如果类本身只有一些基础数据类型，则clone并无问题
- 如果类里面存在着对其他对象的引用，那么拷贝便会将子类的引用拷贝过去，那么便会是不安全的浅层拷贝，并没有完全的独立
- 实现深克隆需要实现Cloneable接口

# 抽象类

1.某些类根本无法初始化,比如说animal类,那类似于这种应该作为一个抽象类abstract,或者说是一个接口   
2.抽象类可以拥有抽象方法和非抽象方法    
3.抽象方法没有实体

------



### 关于引用的一些问题

```
ArrayList<Object> myDogArrayLust = new ArrayList<Object>();   
Dog aDog =new Dog(); 
MyDogArrayList.add(aDog); 
Dog d=myArrayList.get(0);!!!!报错     
```

#### 解析

1.不管放进去了什么,出来也只是Object,除非使用了泛型,标明了类型   
2.传递过程,传递进去一个Dog类型,然后由于参数是Object,父类引用子类,成功add,但是取出的时候, return的参数也是Object    
3.因此说,里面确实存放的是一个包含Dog属性的类型,但是获取的却是一个Object的引用,那么根据多态,子类是无法引用父类的    
4.同样,如果调用一个.eat()方法,那么首先这是一个Object类型的引用,编译器会去寻找Object类及其父类(当然是没有了)的eat()方法,很明显的问题,是不存在的  
5.因为对于编译器来说,当你传递进去的时候,它是它原来的类型,取出来的时候,它确实是一个Object类型,而不是引用了其他类型,编译器只根据引用的类型,而不是对象的类型   

------

- 强制类型转换使用instanceof可以检验是否可以成功转换类型`o instanceof Dog`转换成功返回true

