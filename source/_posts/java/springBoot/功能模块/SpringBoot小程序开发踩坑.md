---
title: SpringBoot小程序开发踩坑
type: tags
tags:
  - null
date: 2018-09-27 11:32:21
categories:
description: 
---
### 接入腾讯云短信

为了给小程序加上短信的业务，腾讯云有100条免费短信，就很舒服地选了腾讯云测试，但是接入的时候报错：

     java-lang-nosuchmethoderror-org-json-jsonobject-getnames-exception

看到日志里面写的是json的处理错误，点开了之后是一个JSON的put报错，本来以为是SDK错了，但是想想，SDK怎么会使得JSON错误呢，找了找问题，才发现是它调用的包和SpringBoot的包冲突了，是源码的问题

# 参考 #
1. [第三方服务腾讯云短信org.json冲突](https://blog.csdn.net/loy_184548/article/details/82707577)
