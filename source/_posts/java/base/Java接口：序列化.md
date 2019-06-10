---
title: Java接口：序列化
type: tags
tags:
  - null
date: 2019-03-21 17:04:33
categories:
description: 序列化
---

# 序列化

- 序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的格式的过程。 
- 反序列化 (Deserialization)是通过从存储或者网络读取对象的状态，重新创建该对象。
- 序列化广泛应用在远程调用（RPC）或者数据存取。

# Serializable接口

Serializable接口，这是一个空接口；

- 如果一个类实现了Serializable接口，那么就代表这个类是自动支持序列化和反序列化的，毋须我们编程实现。
- 如果一个类没有实现Serializable接口，那么默认是不能被序列化的，除非使用其他办法。
- 只能序列化类的状态，不能序列化类的方法。

如果一个类实现了Serializable接口，那么它的子类默认也可以被序列化和反序列化，不论子类是否实现了Serializable接口。

如果一个类实现了Serializable接口，其父类没有实现Serializable接口，那么父类必须有无参的构造器，并且父类中的状态默认不能被序列化。如果想要序列化和反序列化父类，需要在子类里干预序列化的过程，例如使用readObject和writeOjbect方法来序列化和反序列化父类的状态。

## 实例

1. 如何把一个对象序列化到磁盘上，并从磁盘上将该对象恢复，使用的是ObjectOutputStream和ObjectInputStream。
2. 使用transient关键字、readObject和writeOjbect方法、writeReplace和readresolve方法干预序列化。

### 实现序列化

下面的例子演示了如何使用**ObjectOutputStream**把一个Person对象写到磁盘上，之后再通过**ObjectInputStream**把对象恢复。

执行下面的代码可以发现：

- 创建Person对象的时候，Person构造器输出了`I am constructor，too.`；但反序列化构建Person对象的时候，并没有使用构造器。 
- 反序列化回来得到的Person对象p2和之前的p1内地地址不一样，是深拷贝。

```Java
public static void main(String[] args) throws IOException,ClassNotFoundException {
    // Person对象文件路径
    String path = "d:/tmp/person.dat";
    // 创建一个Person对象
    Person p1 = new Person("张三", 18);

    // 序列化Person对象
    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(path));
    out.writeObject(p1);
    out.close();

    // 反序列化
    ObjectInputStream in = new ObjectInputStream(new FileInputStream(path));
    Person p2 = (Person) in.readObject();
    in.close();
    // 打印反序列化的结果
    System.out.println("p2: " + p2);
    // 反序列化后对象的内存地址和原来的地址不同，是深拷贝。
    System.out.println("p1 == p2 is: " + (p1 == p2));
 }
```

Person类，仅有name和age两个属性。

```Java
import java.io.Serializable;

/**
 * 使用java本身的序列化，必须实现Serializable 接口
 */
public class Person implements Serializable {

    private String name;

    private int age;

    public Person() {
        System.out.println("I am constructor");
    }

    public Person(String name, int age) {
        System.out.println("I am constructor，too.");
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

}
```

### 干预序列化

#### 3.1 transient

并不是所有java对象里的内容，都是我们想要序列化写出去的，例如一些隐私数据，比如password等。对于上面的Person对象，假如age是一个隐私字段，我们不想序列化写到磁盘上，那么我们可以用transient关键字来标记它。

```
private transient int age;
```

#### 3.2 readObject和writeOjbect方法

我们可以通过readObject和writeOjbect方法来干预序列化，例如把待序列化的对象的某个属性字段加密和解密，或者把transient关键字标记的属性序列化写出去。

```Java
/**
 * 序列化transient修饰的字段,通过ObjectOutputStream把想要序列化的transient字段写出去。
 * <p>
 * 访问控制符必须是private，一定要先执行out.defaultWriteObject();
 *
 * @param out ObjectOutputStream
 * @throws IOException
 */
private void writeObject(ObjectOutputStream out) throws IOException {
    out.defaultWriteObject();
    out.writeInt(age +10);
}

/**
 * 反序列化transient修饰的字段，通过ObjectInputStream把transient字段读出来
 * <p>
 * 访问控制符必须是private，一定要先执行in.defaultReadObject();
 *
 * @param in ObjectInputStream
 * @throws IOException
 */
private void readObject(ObjectInputStream in) throws IOException, 
                                                ClassNotFoundException {
    in.defaultReadObject();
    age = in.readInt() - 10;
}
```

#### 3.3 readResolve和writeReplace方法

前文已经说过，反序列化得到的对象，和序列化之前的对象，不是同一个对象，它们的内存地址不相同。那么，假设一个单例类实现了Serializable接口，反序列化时内存里就会出现多个实例，这违背了当初设计单例的初衷。

Java的序列化机制提供了一个钩子方法，即私有的readresolve方法，允许我们来控制反序列化时得到的对象。下面的代码就说明了readresolve方法的用法，可以看出反序列化得到的对象，还是唯一的单例对象。

与readresolve方法对应，还有一个writeReplace方法，可以让我们在writeOjbect方法之前修改序列化的对象。

```Java
public class Singleton implements Serializable {

    private Singleton() {
    }

    private static final Singleton INSTANCE = new Singleton();

    public static Singleton getInstance() {
        return INSTANCE;
    }

    private void writeObject(ObjectOutputStream out) throws IOException {
        System.out.println("writeObject");
    }

    private void readObject(ObjectInputStream in) throws IOException,
                                                    ClassNotFoundException {
        System.out.println("readObject");
    }

    /**
     * writeReplace方法会在writeObject方法之前执行。
     * ObjectOutputStream会把writeReplace方法返回的对象序列化写出去。
     * 
     * @return Object
     * @throws ObjectStreamException
     */
    private Object writeReplace() throws ObjectStreamException {
        System.out.println("writeReplace");
        return INSTANCE;
    }

    /**
     * readResolve方法会在readObject方法之后执行，可以再次修改readObject方法返回的对象数据。
     *
     * @return Object
     * @throws ObjectStreamException
     */
    private Object readResolve() throws ObjectStreamException {
        System.out.println("readResolve");
        return INSTANCE;
    }
}
```

验证代码如下，同时也可以看出，序列化时，先执行了writeReplace方法，后执行了writeObject方法；在反序列化的时候，先执行了readObject方法，最后执行了readResolve方法。

```Java
public static void main(String[] args) throws IOException,ClassNotFoundException {
    String path = "d:/tmp/singleton.dat";
    Singleton singleton = Singleton.getInstance();

    // 序列化Singleton对象
    ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(path));
    out.writeObject(singleton);
    out.close();

    // 反序列化
    ObjectInputStream in = new ObjectInputStream(new FileInputStream(path));
    Singleton Singleton2 = (Singleton) in.readObject();
    in.close();

    // 反序列化后对象的内存地址和原来的地址相同。
    System.out.println(singleton == Singleton2);
}
```

输出结果如下：

```Java
readResolve
writeObject
readObject
readResolve
true
```

通常情况下，如无必要，只需要writeObject只和readObject配合使用就够了。此外，关于Serializable，还可能有readObjectNoData()和serialVersionUID需要关注。假如Person对象被序列化到磁盘上了，第二天Person类被修改了，多了旧对象里没有的内容，那么可以实现readObjectNoData，弥补这种因临时扩展而无法兼容反序列化的缺陷。serialVersionUID用于标记对象的版本。

# 拓展部分

上面已经知道可以通过writeObject和readObject、readResolve和writeReplace、还有readObjectNoData方法来干预序列化和反序列化。那么这些方法是什么时候被调用的呢？

他们是在ObjectOutputStream或ObjectInputStream中，配合SerialCallbackContext、ObjectStreamClass类，通过反射回调实现的。ObjectStreamClass类中有invokeWriteObject、invokeReadObject、invokeReadObjectNoData、invokeWriteReplace、invokeReadResolve等方法；有兴趣的同学，可以去看看源码。

以前文的Person类中`private void writeObject(ObjectOutputStream out) throws IOException`的调用为例，我们去ObjectOutputStream类的writeObject()方法开始，依次看下列方法就知道了。

```Java
1.ObjectOutputStream.writeObject(Object obj)
2.ObjectOutputStream.writeObject0(Object obj, boolean unshared)
3.ObjectOutputStream.writeOrdinaryObject(Object obj,
                                 ObjectStreamClass desc,
                                 boolean unshared)
// 就是在第4个方法里初始化了序列化回调上下文SerialCallbackContext
4.ObjectOutputStream.writeSerialData(Object obj, ObjectStreamClass desc)
5.ObjectStreamClass.invokeWriteObject(Object obj, ObjectOutputStream out)
// 6：通过反射调用Person类中的writeObject(ObjectOutputStream out)方法。
6.Method.invoke(Object obj, Object... args)
```

# 参考 #

1. [序列化-Serializable接口](https://www.jianshu.com/p/1d73b49a8a1f)
2. [序列化-Externalizable接口](https://www.jianshu.com/p/47ad7cb1b3bb)
