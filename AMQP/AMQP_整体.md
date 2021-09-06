---
title: "AMQP_整体"
date: 2020-08-18T22:23:07+08:00
tags: ["AMQP"]
categories: ["AMQP"]
keywords: 
- AMQP
draft: false
---

## Message Structure

A message has two parts: a payload and a label (routing key). The payload is the data that you want to transmit. The label (routing key) describes the payload and the RabbitMQ messaging system uses this to determine who will receive a copy of your message.

Unlike TCP — where you need to specify the sender and receiver — AMQP only describes the message with a label. This label includes an exchange name and can optionally include a topic tag. RabbitMQ then sends the label to the interested receivers. This communication is “send-and-forget” and one-directional — see Understanding Messaging: Part One — The Basics for more information on this concept.

## Consumers and Producers

Put simply, producers create messages and consumers receive them. Your application can be a producer when it needs to send a message to another application or it can be a consumer when it needs to receive a message.

### Producers

Producers create messages and publish (send) them to a broker server (in this case, RabbitMQ). Producers create messages and label them for routing.

### Consumers

Consumers attach to a broker server and subscribe to a queue, which is like a named mailbox. Whenever a message arrives in a particular mailbox, RabbitMQ sends it to one of the subscribed/listening consumers. Labels attached to the message are not passed along during routing. As a result, the consumer should only receive one part of the message — the payload. RabbitMQ does not reveal the producer/sender unless the producer includes that message in the message payload. Figure 2 (below) outlines this process.

## Routing a Message

There are three parts to any successful routing of a message in AMQP architecture:

- Exchanges: Where producers publish their message
- Queues: Where consumers receive the message
- Bindings: How the messages are routed from the exchange to a particular queue

### Queues

Queues are like named mailboxes — this is where sent messages wait to be consumed. When a consumer subscribes to a queue, messages are sent immediately to the subscribed consumer.

### Exchanges and Bindings

So how does a message reach a queue? Whenever you want to deliver a message to a consumer, you must first send it to an exchange. Then, based on certain rules or routing keys, RabbitMQ will decide to which queue it should deliver the message.

Rules — or routing keys — enable you to bind queues to exchanges. RabbitMQ will try to match the routing key in the message to those used in the bindings. The message will then be delivered to the queue based on one of three types of exchange: fanout, topic, or direct.

#### Fanout Exchange

This type of exchange will broadcast all of the messages that it receives to all of the queues bound to it. Any routing key provided with the published message will be ignored.

#### Topic Exchange

In this type of exchange, messages are sent to queues based on the routing key. This means that messages sent to a topic exchange must have a specific routing key that must be a list of words, delimited by dots (example, ‘acs.deviceoperations.’). The wording is limited to 255 bytes.

The binding key must be in the same format as the routing key. As a result, a message sent with a particular routing key will be delivered to every queue that is bound with the matching binding key.

The binding key allows for these expression rules:

\* (star) can substitute for exactly one word
\# (hash) can substitute for zero or more words

When a queue is bound with “#” (hash) binding key, it will receive all messages, regardless of the routing key, like in fanout exchange.

#### Direct Exchange

When a queue is declared, it is automatically bound to the exchange that uses the queue name as a routing key. If the routing key matches, then the message is delivered to the corresponding queue.

```
摘自：https://www.incognito.com/tutorials/understanding-messaging-part-two-rabbitmq-2/
```

