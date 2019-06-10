---
title: SpringBoot（一）：注解
type: tags
tags:
  - null
date: 2019-02-28 19:46:22
categories:
description:
---

注解通过实现

```java
//指定注解作用的目标，该注解作用于类
@Target(ElementType.TYPE)
//注解存在的范围
//source：编译被忽略
//class
//runtime：运行时也存在
@Retention(RetentionPolicy.SOURCE)
public @interface ClassName{
    String value() default "";
}
```



# 参考 #

1. 
