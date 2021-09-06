---
title: "AMQP介绍"
date: 2020-08-12T22:14:37+08:00
tags: ["AMQP"]
categories: ["AMQP"]
keywords: 
- AMQP
draft: false
---

### AMQP是什么？

`Advanced Message Queuing Protocol (AMQP) `是一种开放标准协议，面向消息的`应用层`协议。

`AMQP`是一种`应用层`协议，可以让客户端应用程序和消息中间件之间进行通信。



### AMQP

**AMQP**：应用层协议，专注于进程间的通信，是[二进制协议](https://en.wikipedia.org/wiki/Binary_protocol)，因此被认为是紧凑协议，这意味着通过AMQP发送的所有数据都是二进制数据。二进制协议可避免通过网络发送无用的数据。

总体而言，AMQP的目标是通过TCP/IP连接进行消息传递，并保证消息的可靠性，安全性。



### AMQP模型

![amqp.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/amqp/amqp-model.png?raw=true)

```
通常，消息由生产者（producer）产生，并发送给消息代理（broker）【broker包含exchange和queue】，然后交换机（Exchange）根据消息中提供的交换类型和路由键（routing key）将消息发送到队列（queue），最终被消费者（subscriber）使用。
```



### AMQP组成部分



**生产者（Producer）**

```
产生消息
```



**消费者**

```
使用消息
```



##### 消息队列（QUEUE）

```
消息队列（QUEUE）充当缓冲区，用于存储稍后使用的消息。
在创建时，消息队列（QUEUE）可以使用很多属性进行定义。例如，可以将消息队列（QUEUE）标记为持久，自动删除或者互斥，此处互斥是指同一时间只能由一个连接使用该队列，并且在该连接关闭时删除此队列。
```



[**交换机（Exchange）**](https://rabbitmq.mr-ping.com/AMQP/AMQP_0-9-1_Model_Explained.html)

```
交换机（Exchange）是用来发送消息的AMQP实体。交换机（Exchange）拿到一个消息后会将消息给一个或多个消息队列（Queue）。

AMQP定义了消息代理（broker）需要提供四种交换机类型（
	Direct exchange（直连交换机）、
	Fanout exchange（扇型交换机）、
	Topic exchange（主题交换机）、
	Headers exchange（头交换机）
）

交换机（Exchange）有两个状态：持久状态和暂存状态。
	1.持久状态的交换机（Exchange）会在消息代理（broker）重启后依然存在
	2.暂停状态的交换机（Exchange）不会在消息代理（broker）重启后存在
```



**绑定（Binding）**

```
绑定（Binding）是交换机（Exchange）把消息（Message）传给消费队列（QUEUE）遵循的规则

如果要指示交换机“E”把消息交给消息队列“Q”，那么“Q”就需要与“E”进行绑定。绑定操作是由消息中定义的路由键（Routing key）决定交换机（Exchange）绑定到哪个消息队列（QUEUE）。

路由键（Routing key）的意义在于从发送给交换机的众多消息中选择出某些消息，将其路由给绑定的队列。

打个比方：
	队列（queue）是我们想要去的位于纽约的目的地
	交换机（exchange）是JFK机场
	绑定（binding）就是JFK机场到目的地的路线。能够到达目的地的路线可以是一条或者多条
```



**连接（Connection）**

```
AMQP中的连接（Connection）是应用程序和AMQP代理（broker）之间的网络连接，例如TCP/IP套接字连接。
```



**通道（Channels）**

```
有些应用需要与AMQP代理建立多个连接。无论怎样，同时开启多个TCP连接都是不合适的，因为这样做会消耗掉过多的系统资源并且使得防火墙的配置更加困难。AMQP 0-9-1提供了通道（channels）来处理多连接，可以把通道理解成共享一个TCP连接的多个轻量化连接。

在涉及多线程/进程的应用中，为每个线程/进程开启一个通道（channel）是很常见的，并且这些通道不能被线程/进程共享。

一个特定通道上的通讯与其他通道上的通讯是完全隔离的，因此每个AMQP方法都需要携带一个通道号，这样客户端就可以指定此方法是为哪个通道准备的。
```



**虚拟主机（vhost）**

```
虚拟主机（vhost）提供了一种在代理（broker）中隔离应用程序的方法。
不同的用户可以对不同的虚拟主机具有不同的访问权限。当连接被建立的时候，AMQP客户端来指定使用哪个虚拟主机。
```

