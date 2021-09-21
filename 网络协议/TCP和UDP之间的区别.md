# TCP和UDP之间的区别

https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html

##  What is TCP?

[TCP/IP](https://www.guru99.com/tcp-ip-model.html) helps you to determine how a specific computer should be connected to the internet and how you can transmit data between them. It helps you to create a virtual network when multiple computer networks are connected.

TCP/IP stands for Transmission Control Protocol/ Internet Protocol. It is specifically designed as a model to offer highly reliable and end-to-end byte stream over an unreliable internetwork.

In this TCP and UDP difference tutorial, you will learn:

- [What is TCP?](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#1)
- [What is UDP?](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#2)
- [How TCP work?](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#3)
- [How UDP work?](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#4)
- [Features of TCP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#5)
- [Difference between TCP and UDP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#6)
- [Application of TCP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#7)
- [Application of UDP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#8)
- [Advantages of TCP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#9)
- [Advantages of UDP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#10)
- [Disadvantages of TCP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#11)
- [Disadvantages of UDP](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#12)
- [When to use UDP and TCP?](https://www.guru99.com/tcp-vs-udp-understanding-the-difference.html#13)

## What is UDP?

UDP is a Datagram oriented protocol. It is used for broadcast and multicast type of network transmission. The full form of UDP is User Datagram Protocol (A datagram is a transfer unit associated with a packet-switched network.) The UDP protocol works almost similar to TCP, but it throws all the error-checking stuff out, all the back-and-forth communication and deliverability.



## KEY DIFFERENCES:

- TCP is a connection-oriented protocol, whereas UDP is a connectionless protocol.
- The speed for TCP is slower while the speed of UDP is faster
- TCP uses handshake protocol like SYN, SYN-ACK, ACK while UDP uses no handshake protocols
- TCP does error checking and also makes error recovery, on the other hand, UDP performs error checking, but it discards erroneous packets.
- TCP has acknowledgment segments, but UDP does not have any acknowledgment segment.
- When we compare TCP vs UDP protocol, TCP is heavy-weight, and UDP is lightweight.

## How TCP work?

A TCP connection is established with the help of three-way handshake. It is a process of initiating and acknowledging a connection. Once the connection is established, data transfer begins, and when the transmission process is finished, the connection is terminated by the closing of an established virtual circuit.

## How UDP work?

UDP uses a simple transmission method without implied hand-shaking dialogues for ordering, reliability, or data integrity. UDP also assumes that error checking and correction is not important or performed in the application, to avoid the overhead of such processing at the network interface level. It is also compatible with packet broadcasts and multicasting.



## Features of TCP

Here are some important features of TCP:

- Delivery Acknowledgements
- Re transmission
- Delays transmission when the network is congested
- Easy Error detection

Here are some important features of UDP:

- Supports bandwidth-intensive applications that tolerate packet loss
- Less delay
- It sends the bulk quantity of packets.
- Possibility of the Data loss
- Allows small transaction ( DNS lookup)

## Difference between TCP and UDP

Here are the main differences between UDP vs TCP:





![Difference between UDP and TCP](https://cdn.guru99.com/images/1/011720_0714_TCPvsUDPWha1.png)



Difference between UDP and TCP







| TCP                                                          | UDP                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| It is a connection-oriented protocol.                        | It is a connectionless protocol.                             |
| TCP reads data as streams of bytes, and the message is transmitted to segment boundaries. | UDP messages contain packets that were sent one by one. It also checks for integrity at the arrival time. |
| TCP messages make their way across the internet from one computer to another. | It is not connection-based, so one program can send lots of packets to another. |
| TCP rearranges data packets in the specific order.           | UDP protocol has no fixed order because all packets are independent of each other. |
| The speed for TCP is slower.                                 | UDP is faster as error recovery is not attempted.            |
| Header size is 20 bytes                                      | Header size is 8 bytes.                                      |
| TCP is heavy-weight. TCP needs three packets to set up a socket connection before any user data can be sent. | UDP is lightweight. There are no tracking connections, ordering of messages, etc. |
| TCP does error checking and also makes error recovery.       | UDP performs error checking, but it discards erroneous packets. |
| Acknowledgment segments                                      | No Acknowledgment segments                                   |
| Using handshake protocol like SYN, SYN-ACK, ACK              | No handshake (so connectionless protocol)                    |
| TCP is reliable as it guarantees delivery of data to the destination router. | The delivery of data to the destination can’t be guaranteed in UDP. |
| TCP offers extensive error checking mechanisms because it provides flow control and acknowledgment of data. | UDP has just a single error checking mechanism which is used for checksums. |

## Applications of TCP

Here are the applications of TCP:

- It helps you to establish/set up a connection between different types of computers.
- Operates independently of the operating system
- Supports many routing-protocols.
- It enables the internetworking between the organizations.
- It can be operated independently.
- Supports several routing protocols.
- TCP can be used to establish a connection between two computers.

## Applications of UDP

Here are the applications of UDP:

- UDP method is largely used by time-sensitive applications as well as by servers that answer small queries from a larger client base.
- UDP is compatible with packet broadcasts for sending all over the network and for multicasting sending.
- It is also used in Domain Name System, Voice over IP, and online games.

## Advantages of TCP

Here are pros/benefits of TCP:

- It helps you to establish/set up a connection between different types of computers.
- It operates independently of the operating system.
- It supports many routing-protocols.
- It enables the internetworking between the organizations.
- TCP/IP model has a highly scalable client-server architecture.
- It can be operated independently.
- Supports several routing protocols.
- It can be used to establish a connection between two computers.

## Advantages of UDP

Here are the pros/benefits of UDP:

- It never restricts you to a connection-based communication model; that’s why startup latency in distributed applications is low.
- The recipient of UDP packets gets them unmanaged, which also includes block boundaries.
- Broadcast and multicast transmission are also available with UDP
- Data loss can be made
- Small transaction ( DNS lookup)
- Bandwidth intensive app which endures packet loss

## Disadvantages of TCP

Here are disadvantages of using TCP:

- TCP never conclude a transmission without all data in motion being explicitly asked.
- You can’t use for broadcast or multicast transmission.
- TCP has no block boundaries, so you need to create your own.
- TCP offers many features that you don’t want. It may waste bandwidth, time, or effort.
- In this, model the transport layer does not guarantee delivery of packets.
- Replacing protocol in TCP/IP is not easy.
- It doesn’t offer clear separation from its services, interfaces, and protocols.

## Disadvantages of UDP

Here are the cons/drawbacks of UDP:

- In UDP protocol, a packet may not be delivered or delivered twice. It may be delivered out of order, so you get no indication.
- Routers are quite careless with UDP, so they never retransmit it if it collides.
- UDP has no Congestion Control, and flow control, so implementation is the job of a user application.
- UDP mostly like to suffer from worse packet loss

## When to use UDP and TCP?

Now, let’s compare TCP and UDP protocols:

- TCP is an ideal choice, and even it has associated overhead, Therefore, when most of the overhead is in the connection, your application stays connected for any length of time.
- UDP is ideal to use with multimedia like VoIP.
- Use TCP sockets when both client and server independently send packets at that time; an occasional delay is acceptable. (e.g., Online Poker).
- You should use user UDP if both client and server may separately send packets, and occasional delay is also not acceptable. (e.g., Multiplayer games).