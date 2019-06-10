---
title: Spring：MVC
type: tags
tags:
  - Spring
  - 框架
date: 2019-03-26 21:00:39
categories: Java
description:
---

## Spring MVC原理

- 客户端的所有请求都交给前端控制器DispatcherServlet处理，它会负责调用系统的其他模块来真正处理用户的请求
- DispatcherServlet收到请求后，将根据请求的信息（包括URL、HTTP协议方法、请求头、请求参数、cookie等）以及HandlerMapping的配置找到处理该请求的Handler（任何一个对象都可以作为请求的Handler）
- 在这个地方，spring会通过HandlerAdapter对该处理进行封装
- HandlerAdapter是一个适配器，它用统一的接口对各种Handler中的方法进行调用
- Handler完成对用户请求的处理后，会返回一个ModelAndView对象给DispatcherServlet，ModelAndView包含了数据模型以及相应的视图的信息
- ModelAndView的视图是逻辑视图，DispatcherServlet还要借助ViewResolver完成从逻辑视图到真实视图对象的解析工作
- 当得到真正的视图对象后，DispatcherServlet会利用对象对模型数据进行渲染
- 客户端得到响应，可能是一个HTML或json或图片

# 参考

1. 