---
title: 内部类
type: tags
tags:
  - javaBase
date: 2018-10-18 23:16:20
categories: java
description: 内部类，匿名内部类，静态内部类
---
# 内部类

内部类是定义在另一个类当中的类，并且，它与组合是完全不同的概念。

## 分类

内部类有动态与静态之分,动态内部类基于外部类而存在,因此,动态的内部类的创建必须要先创建它的外围类,因为一个动态内部类包含着一个隐含属性,即对其外围类的引用.

而静态内部类并不需要

## 使用原因

- 每个内部类都能独立地继承自一个(接口的)实现,所以无论外部类是否已经继承这个接口的实现,对于内部类没有影响
- **内部类可以实现多重继承**
- 内部类方法可以访问该类定义所在的作用域中的数据，包括私有的数据。(其在创建时候,便与外部类,即创造它的对象建立了一种联系,因此可以访问数据)
- 内部类了解外围类，并能够与其通信
- **利用内部类可以极好的实现private,对外隐藏实现细节**.并且内部类可以对同一个包中的其他类隐藏起来。
- 当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷。

## 使用

内部类可以由外部类的方法进行创建

内部类的对象总会有一个隐式的引用，它指向了创建它的外部对象，因此，内部类是可以直接调用外部类的实例变量

```java
public class InnerClassTest {
    /**
     * 类的方法构造内部类的实例
     */
    public void start() {
        //构造内部类
        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }
    /**
     * 内部类
     * 内部类可以访问外部类的变量
     */
    public class TimePrinter implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + new Date());
            //这个beep是外部类的变量
            if (beep) {
                Toolkit.getDefaultToolkit().beep();
            }
        }
    }
}
```

### 获得内部类的引用

内部类不能在另一个类当中进行创建，若在某些时候想要获取这个引用，需要通过外部类的方法来获得,或者使用.new语法。当然我个人觉得，还不如新建一个类

```java
package ten.innerClass;

/**
 * 在外部获得内部类的引用
 * @Author : Heper
 * @Time : 2019/1/26 16:31
 */
public class InnerClassReference {
    /**
     * 创建内部类
     * 这个内部类必须是一个public的
     */
    public class Content{
        private int i;
        Content(){
            this.i=1;
        }
    }
    public Content getContent(){
        return new Content();
    }
    public static void main(String args[]){
        InnerClassReference innerClassReference=new InnerClassReference();
        //与一般的类创建不同,这个类的创建需要添加它的外围类
        InnerClassReference.Content content=innerClassReference.getContent();
        //也可以采用另一种方式,与平时看到的new语法不同的是,需要在前面加一个外围类的对象
        InnerClassReference.Content content2=innerClassReference.new Content();
    }
}

```

### 与外部类进行交互

#### 操作外部类的元素

```java
package ten.innerClass;


/**
 * 与外部类的链接交互
 * 通过这个内部类,可以实现对数组访问操作的统一,亦或者是对类内部对象、数据等对象的统一操作接口
 * @Author : Heper
 * @Time : 2019/1/26 16:53
 */
public class Sequence {
    private Object[] items;
    private int next = 0;
    public Sequence(int size) { items = new Object[size]; }
    public void add(Object x) {
        if(next < items.length) {
            items[next++] = x;
        }
    }

    /**
     * 这个是内部类,也是一个迭代器
     */
    private class SequenceSelector implements Selector {
        private int i = 0;
        //在该方法当中,直接使用到了外围类的items,不需要任何的中介
        @Override
        public boolean end() { return i == items.length; }
        @Override
        public Object current() { return items[i]; }
        @Override
        public void next() { if(i < items.length) i++; }
    }
    /**
     * 返回一个内部类的引用
     */
    public Selector selector() {
        return new SequenceSelector();
    }
    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for(int i = 0; i < 10; i++) {
            sequence.add(Integer.toString(i));
        }
        //设计模式的思想,只要是继承了selector接口的,都可以使用这个迭代器
        //想要创建一个内部类的对象,必须通过它的外部类来实现
        Selector selector = sequence.selector();
        while(!selector.end()) {
            System.out.print(selector.current() + " ");
            selector.next();
        }
    }
}

/**
 * 一个通用的接口
 */
interface Selector {
    boolean end();
    Object current();
    void next();
}
```

#### 在内部类获得外部类的引用



### 局部内部类

在之前,内部类都是在外部定义的,而事实上可以在一个方法当中或任意的作用域当中定义内部类,这样的内部类称为局部内部类,需要如此的理由是:

- 实现了某类型的接口,于是可以创建并返回对其的引用
- 要解决一个复杂的问题,想创建一个类来辅助解决方案,但并不希望这个类是公共可用的.

在上面，只有start方法使用了内部类，则可以在方法中定义局部类

```java
public class InnerClassTest {
    /**
     * 类的方法构造内部类的实例
     */
    public void start() {
        /**
        * 局部内部类,属于方法,而不属于类,在方法外无法访问
        */
        class TimePrinter implements ActionListener {
            @Override
            public void actionPerformed(ActionEvent event) {
                System.out.println("At the tone, the time is " + new Date());
                //这个beep是外部类的变量
                if (beep) {
                    Toolkit.getDefaultToolkit().beep();
                }
            }
        }
        //构造内部类
        ActionListener listener = new TimePrinter();
        Timer t = new Timer(interval, listener);
        t.start();
    }
}
```

- 不能使用public或者private进行修饰，作用范围只在这个方法当中，对外部完全隐藏。
- 局部方法可以访问局部变量，即在方法当中定义的变量，但是他们必须是不会在方法中改变的，最好是方法的参数

### 匿名内部类

对局部内部类的一个深入，如果只创建这个类的一个对象，则不必命名了。

```java
public class InnerClassTest {
    /**
     * 类的方法构造内部类的实例
     */
    public void start() {
        /**
        * 匿名内部类
        */
          ActionListener listener = new TimePrinter(){
            public void actionPerformed(ActionEvent event) {
                System.out.println("At the tone, the time is " + new Date());
                //这个beep是外部类的变量
                if (beep) {
                    Toolkit.getDefaultToolkit().beep();
                }
            }
        }
        Timer t = new Timer(interval, listener);
        t.start();
    }
}
```
### 静态内部类

如果使用内部类只是为了把一个类隐藏在另外一个类的内部，并不需要内部类引用外围类对象。为此，可以将内部类声明为static,以便取消产生的引用。

# 参考 #
1. 
