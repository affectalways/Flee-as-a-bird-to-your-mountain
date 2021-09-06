---
title: "AMQP_消息"
date: 2020-08-18T22:22:07+08:00
tags: ["AMQP"]
categories: ["AMQP"]
keywords: 
- AMQP
draft: false
---

## What is Messaging?

Today, real-time information is constantly required and available. This  needs easy ways to be routed to multiple receivers reliably and quickly. Integration solutions have to deal with a few fundamental challenges:

- **Networks are unreliable.** Integration solutions have to transport data from one computer to another across networks, causing delays or interruptions.
- **Networks are slow.** Sending data across a network is much slower than making a local call.
- **Any two applications are different.** Integration solutions need to transmit information between systems that use different programming languages, operating platforms, and data formats. An integration solution needs to be able to interface with all these different technologies.
- **Applications change over time.** An integration solution has to keep pace with changes in the applications that it connects. It should also minimize the dependencies from one system to another by using loose coupling between applications.

Messaging is a type of technology that helps you overcome the above challenges through asynchronous, program-to-program communication. Messaging enables software applications to connect and scale by separating the sending and receiving of data. There are several important concepts to understand:

- Programs communicate by sending packets of data (messages) to each other.
- Channels (or queues) are logical pathways that connect the programs and convey messages.
- A sender or producer is a program that sends a message by writing the message to a channel.
- A receiver or consumer is a program that receives a message by reading it from a channel.

### Important Messaging Concepts

- **Send and forget.** The sending application sends the message to the message channel (queue). Once that step is complete, the sender can move onto other work while the messaging system transmits the message in the background. The sender does not have to wait for the consumer to receive and process the message.
- **Store and forward.** The messaging system stores the message, either in memory or on disk, and delivers the message to the receiver’s computer.

## Why Use a Messaging System?

Messaging capabilities are provided by a separate software system called a messaging system. A messaging system manages the channels that define the paths of communication between the applications and the sending and receiving of messages. The  main task of a messaging system is to reliably move messages from the sender’s computer to the receiver’s computer.

The benefits of messaging include:

- **Remote communication.** Messaging enables separate applications to communicate and transfer data.
- **Platform/language integration.** When connecting multiple computer systems via remote communication, it’s likely that these systems use different languages, technologies, and platforms. A messaging system allows these disparate to integrate.
- **Asynchronous communication.** Messaging enables a “send and forget” approach to communication. The sender does not have to wait for the receiver to receive and process the message.
- **Variable timing.** With synchronous communication, the caller must wait for the receiver to finish processing the call before the caller can receive the result and continue. This means that the caller can only make calls as fast as the receiver can perform them. Conversely, asynchronous communication allows the sender to batch requests to the receiver at its own pace and for the receiver to consume the requests at its own different pace.
- **Reliable communication.** The messaging uses a “store and forward” approach to transmitting messages. The data is packaged as messages, which are atomic, independent units. When the sender sends a message, the messaging system stores the message and then delivers it by forwarding the message to the receiver’s computer.
- **Topic-based messages.** Receivers can register to consume messages selectively based on particular topics of interest.

[RabbitMQ](http://www.rabbitmq.com/) is an asynchronous messaging system that can be be used to allow disparate applications share data via a common protocol, [Advanced Message Queuing Protocol (AMQP)](http://amqp.org/). This protocol was designed to be an open standard that would solve the vast majority of messaging needs and topologies. RabbitMQ is the messaging system used in Incognito Auto Configuration Server.

## Further Reading

For more information on message components and how the processes work in RabbitMQ, refer to Understanding Messaging: Part Two — RabbitMQ.



```
摘自：https://www.incognito.com/tips-and-tutorials/understanding-messaging-part-one-the-basics-2/
```

