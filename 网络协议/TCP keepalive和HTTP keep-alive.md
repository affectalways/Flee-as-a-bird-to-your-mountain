### TCP 的 keepalive 了解吗？说一说它和 HTTP 的 keep-alive 的区别？

### TCP keepalive

#### 概念

在使用TCP长连接（复用已建立TCP连接）的场景下，需要对TCP连接进行保活，避免被网关干掉连接。
在应用层，可以通过定时发送心跳包的方式实现。而Linux已提供的TCP KEEPALIVE，在应用层可不关心心跳包何时发送、发送什么内容，由OS管理：OS会在该TCP连接上定时发送探测包，探测包既起到**连接保活**的作用，也能自动检测连接的有效性，并**自动关闭无效连接**。



#### 原理

建立TCP连接时，就有定时器与之绑定，其中的一些定时器就用于处理keepalive过程。当keepalive定时器到0的时候，便会给对端发送一个不包含数据部分的keepalive探测包（probe packet），如果收到了keepalive探测包的回复消息，那就可以断定连接依然是OK的。如果我们没有收到对端keepalive探测包的回复消息，我们便可以断定连接已经不可用，进而采取一些措施。但Keepalive会额外产生一些网络数据包外，这些包将加大网络流量，对路由器和防火墙造成一定的负担。



#### Linux设置

```
# cat /proc/sys/net/ipv4/tcp_keepalive_time
7200
# cat /proc/sys/net/ipv4/tcp_keepalive_intvl
75
# cat /proc/sys/net/ipv4/tcp_keepalive_probes
9
```

tcp_keepalive_time，在TCP保活打开的情况下，如果在该时间内没有数据往来，则发送探测包。即允许的持续空闲时长，或者说每次正常发送心跳的周期，默认值为7200s（2h）。
tcp_keepalive_probes 尝试探测的次数。如果发送的探测包次数超过该值仍然没有收到对方响应，则认为连接已失效并关闭连接。默认值为9（次）
tcp_keepalive_intvl，探测包发送间隔时间。默认值为75s



### 更多细节

#### 长连接环境为什么要保活

TCP连接被中间设备断开，客户端和服务端对此是得不到通知的。长连接的环境下，如IM服务，若某连接长时间没有数据交换，可能被丢弃。那一旦有数据需要传递，且此时连接已经被中介设备断开，应用程序没有及时感知的话，那么就会导致在一个无效的数据链路层面发送业务数据，结果就是发送失败。



#### TCP连接不活跃被干掉具体是什么

在三层地址转换中，我们可以保证局域网内主机向公网发出的IP报文能顺利到达目的主机，但是从目的主机返回的IP报文却不能准确送至指定局域网主机（我们不能让网关把IP报文广播至全部局域网主机，因为这样必然会带来安全和性能问题）。为了解决这个问题，网关路由器需要借助传输层端口，通常情况下是TCP或UDP端口，由此来生成一张**传输层端口转换表**。
由于**端口数量有**限（0~65535），端口转换表的维护占用系统资源，因此不能无休止地向端口转换表中增加记录。对于过期的记录，网关需要将其删除。如何判断哪些是过期记录？网关认为，一段时间内无活动的连接是过期的，应**定时检测转换表中的非活动连接，并将之丢弃**。

更多参考：[https://blog.chionlab.moe/201...](https://link.segmentfault.com/?url=https%3A%2F%2Fblog.chionlab.moe%2F2016%2F09%2F24%2Flinux-tcp-keepalive%2F)





### Http keep-alive

在 HTTP 1.0 时期，每个 TCP 连接只会被一个 HTTP Transaction（请求加响应）使用，请求时建立，请求完成释放连接。当网页内容越来越复杂，包含大量图片、CSS 等资源之后，这种模式效率就显得太低了。所以，在 HTTP 1.1 中，引入了 HTTP persistent connection 的概念，也称为 HTTP keep-alive，目的是复用TCP连接，在一个TCP连接上进行多次的HTTP请求从而提高性能。

HTTP1.0中默认是关闭的，需要在HTTP头加入"Connection: Keep-Alive"，才能启用Keep-Alive；HTTP1.1中默认启用Keep-Alive，加入"Connection: close "，才关闭。



#### 区别

两者在写法上不同，http keep-alive 中间有个"-"符号。
HTTP协议的keep-alive 意图在于**连接复用**，同一个连接上串行方式传递请求-响应数据
TCP的keepalive机制意图在于**保活**、心跳，检测连接错误。





### http连接池与keep-alive关系

连接池技术作为创建和管理连接的缓冲池技术，减少创建和销毁连接的开销，是对多个连接的生命周期管理。keep-alive强调的是一个http连接本身的复用机制。



### 总结

本文重点梳理了TCP keep alive概念和原理，并与http keep-alive对比区别，又引申出http连接池与keep-alive的关系，帮助理清基础概念。





### 参考

https://segmentfault.com/a/1190000012894416

https://link.segmentfault.com/?url=https%3A%2F%2Fblog.chionlab.moe%2F2016%2F09%2F24%2Flinux-tcp-keepalive%2F

https://link.segmentfault.com/?url=http%3A%2F%2Ftime-track.cn%2Ftcp-keepalive-howto.html
https://link.segmentfault.com/?url=http%3A%2F%2Fwww.blogjava.net%2Fyongboy%2Farchive%2F2015%2F04%2F14%2F424413.html
https://link.segmentfault.com/?url=http%3A%2F%2Fwww.cnblogs.com%2Fzhanjindong%2Fp%2Fhttpclient-connection-pool.html