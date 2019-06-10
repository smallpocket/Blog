---
title: java枚举类
type: tags
tags:
  - JavaBase
date: 2018-10-08 14:35:04
categories: java
description: 枚举的创建于作用，使用枚举管理状态码
---
# 基础

## 如何构造一个枚举类

1. 新建一个Enum的类
2. 添加枚举的实例,实例的要求:
- 名称要大写
- 多个单词是使用下划线进行分割
- 实例之间使用逗号分割
- 最后实例结尾使用分号;
3. 为类添加字段
4. 必须添加构造方法
5. 可选:
- 为字段添加getter和setter
- 为实例添加doc注释

## 代码
``` java
public enum SimpleEnum {
    /**
     * 为枚举添加注释
     */
    TEST("1"),
    ;
    /**
     * 枚举类需要字段和构造器两个属性,否则会出错
     */
    private String code;

    public String getCode() {
        return code;
    }

    SimpleEnum(String code) {
        this.code = code;
    }
}
```

## 枚举的作用

### 管理code或者一些常量

在实际项目当中,会遇到一些状态码的相关东西,这个使用枚举进行维护是相对便捷的

### 进行switch

``` java
    /**
     * 使用switch和枚举类进行结合
     */
    public void degree(){
        SimpleEnum simpleEnum=SimpleEnum.TEST;
        switch (simpleEnum){
            case TEST:
                System.out.println("test");
                return;
            case ORDER:
                System.out.println("orde");
                return;
                default:
                    System.out.println("ss");
        }
    }
```

# 参考 #
1. 
