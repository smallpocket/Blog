---
title: Redis（2）：数据结构与对象
type: tags
tags:
  - null
date: 2019-03-04 18:34:18
categories:
description:
---

# 数据结构与对象

Redis数据库里面的每个键值对都是由对象组成的

- 数据库键总是一个字符串对象
- 数据库键的值可以是
  - 字符串对象
  - 列表对象
  - 哈希对象
  - 集合对象（set）
  - 有序集合对象

# 简单动态字符串SDS

Redis的字符串与传统字符串（以空字符结尾的字符数组，C字符串）不同，Redis自己构建了一种简单动态字符串（SDS）的抽象类型，并用作默认字符串表示。

- 传统字符串在Redis当中只会作为字符串字面量用在一些无须对字符串进行修改的地方。
- 当需要一个可以被修改的字符串值时，就会使用SDS来表示字符串值

SDS使用

- 保存数据库的字符串值
- 用作缓冲区
  - AOF模块中的AOF缓冲区
  - 客户端状态中的输入缓冲区

## SDS定义

```
struct sdshdr{
//记录buf数组已经使用的字节的数量
    int len;
    //记录未被使用字节的数量
    int free;
    //保存字符串
    //最后一个字节保存空字符'\0'，不记在len当中
    char buf[];
}
```

## 与C字符串的区别

C字符串：以长度N+1的字符数组来表示长度为N的字符串，并且字符数组的最后一个元素总是空元素'\0'

C字符串的字符串表示方法不能满足Redis对字符串在安全性、效率以及功能方面的要求

- C字符串需要常数复杂度获取字符串长度。
  - 它不记录自身长度，程序必须遍历整个字符串才能获取长度
- C字符串容易造成缓冲区溢出。
  - C语言字符串拼接函数strcat，假定用户在执行函数时分字符串分配了足够多的内存，如果内存不够，则会缓冲区溢出
    - 这是因为字符串本身不记录其本身的内存信息
  - SDS有free与len，可以知道自身剩余内存大小
- 减少修改字符串时带来的内存重分配次数
  - C字符串每次修改时，都要对保存字符串的数组进行一次内存重分配操作
    - 如果扩展字符串，则需要先分配内存；如果截断，则需要释放不适用的空间，否则会内存泄露。
    - 也因此需要进行系统调用，较为费时
  - SDS空间预分配优化策略
    - 优化字符串增长操作，当需要对SDS进行空间扩展时，程序不仅会为SDS分配必要空间，还会为SDS分配额外的未使用空间
      - 如果SDS修改后，len<1MB，则程序分配和len同样大小的未使用空间
      - 如果len>=1MB，则分配1MB的未使用空间
  - SDS惰性空间释放优化策略
    - 优化缩短操作，程序并不立即使用内存重分配来回收多出来的字节，而是使用free记录下来，等待将来使用
    - 同时提供了相应API，在需要时刻真正释放空间
- 二进制安全
  - C字符串的字符必须包含某种编码，并且不能有空字符。因此只能保存文本，不能保存图片等二进制数据
  - SDS的API都以处理二进制的方式来处理SDS里的数据
- 兼容**部分**C字符串函数

## SDS的主要操作API

![1551695057543](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695057543.png)

![1551695070215](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695070215.png)

# 链表

## 应用

- 列表键的底层实现之一
  - 当一个列表键包含了数量较多的元素，或者列表中包含的元素都是比较常的字符串时，会使用链表作为链表键的底层实现
- 发布与订阅
- 慢查询
- 监视器
- 保存多个客户端的状态信息
- 构建客户端输出缓冲区

## 实现

```C
typedef struct listNode{
	
    struct listNode *prev;
    struct listNode *next;
    //节点的值
    void *value;
}listNode;

typedef struct list{
	
    listNode *head;
    listNode *tail;
    //长度
    unsigned long len;
    //节点值赋值函数 dup
    void *(*dup)(void *ptr);
    //节点释放函数 free
    void *(*free)(void *ptr);
    //节点值对比函数 match
    void *(*match)(void *ptr,void *key);;
}list;
```

特性

- 双端
- 无环
  - 表头的prev与表尾的next都指向null
- 表头指针与表尾指针
- 长度计数器
- 多态
  - void*保存节点值，可以保存各种不同类型的值
  - 使用三个属性可以为节点值设置类型特定函数

## API

![1551695605130](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695605130.png)

![1551695615751](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1551695615751.png)

# 字典

map

## 应用

- 哈希键的底层实现之一，当一个哈希键包含的键值对比较多，或者键值对中的元素都是比较长的字符串时。
- 当字典用作数据库底层实现或哈希键的底层实现时，使用MurmurHash2算法计算键的哈希值
  - 即使输入的键有规律，算法依然能给出一个很好地随机分布性，并且计算速度也很快

## 实现

使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，每个哈希表节点就保存了字典中的一个键值对

**哈希表**

```C
//字典
//type与privdata属性针对不同类型的键值对，创建动态字典而设置
//dictType保存了一族用于操作特定类型键值对的函数
//privdata保存了需要传给那些类型特定函数的可选参数
typedef struct dict{
    //类型特定函数
    dictType *type;
    //私有数据
    void *privdata;
    //哈希表
    //一般使用ht[0],ht[1]在rehash时使用
    dictht ht[2];
    //rehash索引
    //当rehash不在进行时，值为-1
    in trehashidx;
}dict;

typedef struct dictType{
    //计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);
    //复制键的函数
    void *(*keyDup)(void *privdata,const void *key);
    //复制值的函数
    void *(*valDup)(void *privdata,const void *obj);
    //对比键的函数
    int (*keyCompare)(void *privdata,const void *key1,const void *key2);
    //销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);
    //销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);
}

//哈希表
typedef struct dictht{
	//哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;
    //哈希表大小掩码，用于计算索引值
    //总是等于size-1
    //与哈希值一起决定一个键应该放到table的哪个索引上
    unsigned long sizemask;
    //已有节点数量
    unsigned long used;
}dictht;

//哈希表节点
typedef struct dictEntry{
    void *key;
    //值
    union{
        void *val;
        uint_tu64;
        int64_ts64;
    }v;
    //解决冲突
    strut dictEntry *next；
}dictEntry;
```

## 哈希算法

- 根据键值对的键计算出哈希值和索引值
- 根据索引值，将包含新键值对的哈希表节点放到哈希表数组的制定索引上
- index=hash&dict->ht[x].sizemask;

## 解决键冲突

链地址法，并且将新节点加到表头，则复杂度为O1

## rehash

随着操作不断执行，哈希表的键值对数目变化，为了保持负载因子维持在一个合理范围内，对哈希表进行相应的扩展或者收缩。

**步骤**

- 为ht[1]哈希表分配空间，大小取决于要执行的操作，以及ht[0]当前包含的键值对数量，即used值
  - 如果扩展，则大小为第一个>=ht[0].used*2的2^n
  - 如果收缩，则大小为第一个>=ht[0].used的2^n
- 将保存在ht[0]的所有键值对rehash到ht[1]上。
  - rehash指重新计算键的哈希值与索引值
- 释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]创建一个空白哈希表

扩展：

- 服务器没有执行BGSAVE或BGREWRITEAOF命令，并且哈希表负载因子>=1
- 服务器在执行BGSAVE或BGREWRITEAOF命令，并且哈希表负载因子>=5
  - 执行命令过程中，Redis需创建当前服务器进程的子进程
  - 大多数OS采用**写时复制**技术优化子进程的使用效率
    - 父进程和子进程共享页面而不是复制页面。然而，只要页面被共享，它们就不能被修改。
    - 无论父进程和子进程何时试图写一个共享的页面，就产生一个错误，这时内核就把这个页复制到一个新的页面中并标记为可写。原来的页面仍然是写保护的：
    - 当其它进程试图写入时，内核检查写进程是否是这个页面的唯一属主；如果是，它把这个页面标记为对这个进程是可写的。
  - 子进程存在期间，服务器会提高负载因子，以尽可能避免在子进程存在期间进行扩展，避免不必要的内存写入操作，节约内存

收缩：当负载因子小于0.1时

## 渐进式rehash

在哈希表内保存极大量键值对时，一次性rehash会导致服务器停止服务。因此分多次、渐进式rehash

- 为ht[1]哈希表分配空间
- 在字典中维持一个索引计数器变量rehashidx标识rehash开始
- rehash进行期间，每次对字典执行增删改查操作时，除了执行指定操作，还会将ht[0]在rehashidx索引上的键值对rehas到ht[1]，在rehash完成后，rehashidx自增
- 全部rehash结束，rehashidx=-1
- 查找时，会先查ht[0]后查ht[1]。

## API

![1552026275087](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552026275087.png)

![1552026286675](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552026286675.png)

# 跳跃表

## 应用

- 一种有序数据结构，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。
- 支持平均OlgN,最坏ON的节点查找，可以通过顺序性操作批处理节点
- 大部分情况下，效率与平衡树媲美，并且实现简单。
- 有序集合键的底层实现之一，如果一个有序集合包含的元素量比较多，或者元素的成员是比较长的字符串时。
- 集群节点中用作内部数据结构

## 实现

```C
typedef struct zskiplist{
    //表头表尾
    structz skiplistNode *header,*tail;
    //表中节点数量
    unsigned long length;
    //表中层数最大的节点的层数
    int level;
}

typedef struct zskiplistNode{
    //层
    //包含多个元素，每个元素都包含一个指向其他节点的指针，可以通过层加快访问其他节点的速度
    //层越多，访问其他节点速度越快
    //创建一个新的节点时，随机生成一个1-32的值，作为层数
    struct zskiplistLevel{
        //前进指针
        //用于访问位于表尾方向的其他节点
        struct zskiplistNode *forward;
        //跨度
        //前进指针所指向节点和当前节点的距离
        unsigned int span;
    }level[];//是一个数组
    
    //后退指针
    //当前节点的前一个节点
    struct zskiplistNode *backward;
    //分值
    //节点按各自保存的分值从小到大排列
    double score;
    //成员对象，一个SDS，较小的排前面，较大排后面（表尾）
    //成员对象必须唯一，但是分值可以相同
    robj *obj;
}zskiplistNode;
```

## 查找

![1552029291744](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552029291744.png)

## API

![1552029490765](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552029490765.png)

# 整数集合

## 应用

- 集合键的底层实现之一。当一个集合只包含整数值元素，并且这个结合数量不多。

## 实现

```C
typedef struct intset{
	//编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    //从小到大有序排列，并不包含重复项
    //真正类型取决于encoding的值
    int8_t contents[];
}intset;
```

当向一个int16_t的数组，添加一个int64_t的数值时，所有元素都会被转换为int64_t。

## 升级

当要将一个新元素添加到整数集合里，并且新元素的类型比整数集合现有所有元素类型都长时，整数集合需要先进行升级，然后才能添加元素。

- 根据新元素的类型，扩展整数集合底层数组的空间，并为新元素分配空间
- 将底层数组现有的所有元素转换成新元素的类型，并将转换后的元素放置到正确的位置，且维持有序性
- 将新元素添加进去

不支持降级操作

**优势**

- 提升灵活性，不担心类型错误
- 节约内存

## API

![1552030303322](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552030303322.png)

# 压缩列表

## 应用

- 列表键与哈希键的底层实现之一。当一个列表键只包含少量列表项，并且每个列表项要么就是小整数值，要么就是长度较短的字符串

## 实现

- Redis为了节约内存而开发，由一系列特殊编码的连续内存块组成的顺序性数据结构。
- 一个压缩列表可以包含多个节点，每个节点可以保存一个字节数或者一个整数值

**构成**

- zibytes：记录整个压缩列表占用的内存字节数
- zltail：记录压缩列表表尾节点距离首部的偏移
- zllen：节点数目
- entry[]：包含的节点
- zlend：特殊值，标记末端

**节点构成**

- previous_entry_length：记录前一个节点的长度（字节为单位）
- encoding：记录content属性所保存数据的类型已经长度
- content：保存节点的值

## 连锁更新

当新插入或删除的节点导致相邻的节点的previous_entry_length属性所占据的空间无法保存正确的数据，而导致需要空间分配。并且出现连锁反应

- 实际触发可能性较低，并且更新的节点一般不会太多，因此不存在性能问题

## API

![1552033129986](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552033129986.png)

# 对象

- Redis基于数据结构创建了一个对象系统，而不是直接实现数据库。包括字符串对象、列表对象、哈希对象、集合对象、有序集合对象。
- 在执行命令前，可以根据对象的类型判断对象是否可以执行给定的命令
- 可以针对不同的使用场景，为对象设置多种不同的数据结构实现，从而优化使用效率
- 实现了基于引用计数的内存回收机制、对象共享共享机制（多个数据库键共享同一个对象来节约内存）
- 对象带有访问时间记录信息，可以用于计算数据库键的空转时长，在服务器启用maxmemory下，空转时长较大的键可能被服务器优先删除

## 对象的类型与编码

Redis使用对象来表示数据库中的键和值，每次创建一个键值对，则会至少创建两个对象：键对象、值对象

对象的实现：

```C
typedef struct redisObject{
    //类型
    unsigned type:4;
    //编码
    unsigned encoding:4;
    //指向底层实现数据结构的指针
    void *ptr;
    //引用计数
    int refcount;
    //对象最后一次被命令程序的访问的时间
    unsigned lru:22;
    //....
}
```

**类型**

![1552033987495](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552033987495.png)

键总是一个字符串对象，而值可能是其中一种。

字符串键：则指该键对应的值为字符串对象

**编码和底层实现**

ptr指向对象的底层实现数据结构，而数据结构由encoding属性决定

![1552034120684](C:\Users\Heper\AppData\Roaming\Typora\typora-user-images\1552034120684.png)

- 根据不同使用场景为对象设置不同的编码，优化对象效率
- 在列表对象包含元素较少，则使用压缩列表。更节约内存，在内存中连续块保存更快载入到内存
- 元素越多时，双端链表更优

## 字符串对象

- int
- raw
- embstr
  - embstr专门用于保存短字符串的一种优化编码方式。raw编码会调用两次内存分配函数创建redisObject与sdshdr，而embstr调用一次内存分配函数来分配一块连续空间，能更好地利用缓存带来的优势。

## 列表对象

- ziplist
- linkedlist

## 哈希对象

- ziplist
  - 当有新的键值对加入，先将保存了键的压缩列表节点推入到表尾，再将保存了值的节点推入表尾
    - 两个节点总是紧挨，键节点在前，值节点在后
    - 先添加到哈希对象中的键值对放在表头方向
- hashtable
  - 字典作为底层实现

## 集合对象

- intset
  - 整数集合
- hashtable
  - 字典

## 有序集合对象

- ziplist
- skiplist

## 类型检查与命令多态

操作键的命令可分为两种类型

- 可对任何类型的键执行
  - DEL、EXPIRE、RENAME、TYPE、OBJECT
- 只能对特定类型的键执行
  - SET、GET等只能对字符串键执行
  - HDEL、HSET等只能对哈希键执行
  - ...

**类型检查的实现**

在执行一个类型特定的命令前，Redis会先检查redisObject的type属性

**多态命令的实现**

根据值对象的编码方式，选择正确的命令实现代码来执行命令

- 根据类型的多态 
  - DEL等
- 根据编码的多态
  - SET等，对于一种类型的多种实现编码

## 内存回收

根据redisObject的refcount属性记录

- 当创建一个新对象时，引用计数的值初始化为1
- 当被一个新程序使用时，引用计数+1
- 变为0时，占用内存会释放

## 对象共享

当键A与键B都创建了一个包含整数值100的字符串对象作为值对象

- 为键B创建一个新的对象
- 键A与键B共享一个对象
  - 将键B的指针指向原有的对象
  - 被共享值的对象引用计数+1

Redis只对包含整数值的字符串对象进行共享，因为对于包含字符串的话，做一个equal的时间复杂度太高，CPU占用时间太长

## 对象的空转时长

依据对象的lru时间，如果占用内存超出maxmemory则lru较高的部分键会优先被释放

# 参考

1. 《Redis设计与实现》

