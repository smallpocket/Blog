---
title: MQTT协议
type: tags
tags:
  - null
date: 2019-05-29 11:58:05
categories:
description:
---

## 提出问题

## 为什么要用（作用）

## 应用场景

## 基础概述

### 是什么

**MQTT**[[1\]](https://zh.wikipedia.org/wiki/MQTT#cite_note-1)(**消息队列遥测传输**)是[ISO 标准](https://zh.wikipedia.org/wiki/%E5%9B%BD%E9%99%85%E6%A0%87%E5%87%86%E5%8C%96%E7%BB%84%E7%BB%87)(ISO/IEC PRF 20922)[[2\]](https://zh.wikipedia.org/wiki/MQTT#cite_note-ISO-2)下基于[发布/订阅](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85)范式的消息协议。它工作在 [TCP/IP协议族](https://zh.wikipedia.org/wiki/TCP/IP%E5%8D%8F%E8%AE%AE%E6%97%8F)上，是为硬件性能低下的远程设备以及网络状况糟糕的情况下而设计的[发布/订阅](https://zh.wikipedia.org/wiki/%E5%8F%91%E5%B8%83/%E8%AE%A2%E9%98%85)型消息协议，为此，它需要一个[消息中间件](https://zh.wikipedia.org/w/index.php?title=%E6%B6%88%E6%81%AF%E4%B8%AD%E9%97%B4%E4%BB%B6&action=edit&redlink=1) 。

MQTT是一个基于客户端-服务器的消息发布/订阅传输协议。MQTT协议是轻量、简单、开放和易于实现的，这些特点使它适用范围非常广泛。在很多情况下，包括受限的环境中，如：机器与机器（M2M）通信和物联网（IoT）。其在，通过卫星链路通信传感器、偶尔拨号的医疗设备、智能家居、及一些小型化设备中已广泛使用。

![img](assets/mqtt-fidge-2.svg)

### 分类，各个分类是什么

## 基础

### 优缺

优点

- 可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用。

### 实现

#### 实现步骤

#### 示例

### 协议原理

#### MQTT协议实现方式

实现MQTT协议需要客户端和服务器端通讯完成，在通讯过程中，MQTT协议中有三种身份：发布者（Publish）、代理（Broker）（服务器）、订阅者（Subscribe）。其中，消息的发布者和订阅者都是客户端，消息代理是服务器，消息发布者可以同时是订阅者。

MQTT传输的消息分为：主题（Topic）和负载（payload）两部分：

- Topic，可以理解为消息的类型，订阅者订阅（Subscribe）后，就会收到该主题的消息内容（payload）；
- payload，可以理解为消息的内容，是指订阅者具体要使用的内容。

#### 网络传输与应用消息

MQTT会构建底层网络传输：它将建立客户端到服务器的连接，提供两者之间的一个有序的、无损的、基于字节流的双向传输。

当应用数据通过MQTT网络发送时，MQTT会把与之相关的服务质量（QoS）和主题名（Topic）相关连。

#### MQTT客户端

一个使用MQTT协议的应用程序或者设备，它总是建立到服务器的网络连接。客户端可以：

- 发布其他客户端可能会订阅的信息；
- 订阅其它客户端发布的消息；
- 退订或删除应用程序的消息；
- 断开与服务器连接。

#### MQTT服务器

MQTT服务器以称为"消息代理"（Broker），可以是一个应用程序或一台设备。它是位于消息发布者和订阅者之间，它可以：

- 接受来自客户的网络连接；
- 接受客户发布的应用信息；
- 处理来自客户端的订阅和退订请求；
- 向订阅的客户转发应用程序消息。

#### MQTT协议中的订阅、主题、会话

**一、订阅（Subscription）**

订阅包含主题筛选器（Topic Filter）和最大服务质量（QoS）。订阅会与一个会话（Session）关联。一个会话可以包含多个订阅。每一个会话中的每个订阅都有一个不同的主题筛选器。

**二、会话（Session）**

每个客户端与服务器建立连接后就是一个会话，客户端和服务器之间有状态交互。会话存在于一个网络之间，也可能在客户端和服务器之间跨越多个连续的网络连接。

**三、主题名（Topic Name）**

连接到一个应用程序消息的标签，该标签与服务器的订阅相匹配。服务器会将消息发送给订阅所匹配标签的每个客户端。

**四、主题筛选器（Topic Filter）**

一个对主题名通配符筛选器，在订阅表达式中使用，表示订阅所匹配到的多个主题。

**五、负载（Payload）**

消息订阅者所具体接收的内容。

#### MQTT协议中的方法

MQTT协议中定义了一些方法（也被称为动作），来于表示对确定资源所进行操作。这个资源可以代表预先存在的数据或动态生成数据，这取决于服务器的实现。通常来说，资源指服务器上的文件或输出。主要方法有：

- Connect。等待与服务器建立连接。
- Disconnect。等待MQTT客户端完成所做的工作，并与服务器断开TCP/IP会话。
- Subscribe。等待完成订阅。
- UnSubscribe。等待服务器取消客户端的一个或多个topics订阅。
- Publish。MQTT客户端发送消息请求，发送完成后返回应用程序线程。

### 设计思想

MQTT协议工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议，它具有以下主要的几项特性：

- 使用发布/订阅消息模式，提供一对多的消息发布，解除应用程序耦合。

  这一点很类似于XMPP，但是MQTT的信息冗余远小于XMPP，,因为XMPP使用XML格式文本来传递数据。

- 对负载内容屏蔽的消息传输。

- 使用TCP/IP提供网络连接。

  主流的MQTT是基于TCP连接进行数据推送的，但是同样有基于UDP的版本，叫做MQTT-SN。这两种版本由于基于不同的连接方式，优缺点自然也就各有不同了。

- 有三种消息发布服务质量：

  "至多一次"，消息发布完全依赖底层TCP/IP网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。这一种方式主要普通APP的推送，倘若你的智能设备在消息推送时未联网，推送过去没收到，再次联网也就收不到了。

  "至少一次"，确保消息到达，但消息重复可能会发生。

  "只有一次"，确保消息到达一次。在一些要求比较严格的计费系统中，可以使用此级别。在计费系统中，消息重复或丢失会导致不正确的结果。这种最高质量的消息发布服务还可以用于即时通讯类的APP的推送，确保用户收到且只会收到一次。

- 小型传输，开销很小（固定长度的头部是2字节），协议交换最小化，以降低网络流量。

  这就是为什么在介绍里说它非常适合"在物联网领域，传感器与服务器的通信，信息的收集"，要知道嵌入式设备的运算能力和带宽都相对薄弱，使用这种协议来传递消息再适合不过了。

- 使用Last Will和Testament特性通知有关各方客户端异常中断的机制。

  Last Will：即遗言机制，用于通知同一主题下的其他设备发送遗言的设备已经断开了连接。

  Testament：遗嘱机制，功能类似于Last Will。

## 进阶

## 反省总结

# 参考 #

1. [维基百科](<https://zh.wikipedia.org/wiki/MQTT>)
