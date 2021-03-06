# A、B 机器正常连接后，B 机器突然重启，问 A 此时处于 TCP 什么状态

## 问题定义

- A -> B 发起TCP请求，A端为请求侧，B端为服务侧
- TCP 三次握手已完成
- TCP 三次握手后双方没有任何数据交互
- B 在无预警情况下掉线（类似意外掉电重启状态）

## 问题答案

### 结论

A侧的TCP链路状态在未发送任何数据的情况下与等待的时间相关，如果在多个超时值范围以内那么状态为<**established**>;如果触发了某一个超时的情况那么视情况的不同会有不同的改变。

一般情况下不管是KeepAlive超时还是内核超时，只要出现超时，那么必然会抛出异常，只是这个异常截获的时机会因编码方式的差异而有所不同。（同步异步IO，以及有无使用select、poll、epoll等IO多路复用机制）

### 原因与相关细节

<**大前提**>

基于IP网络的无状态特征，A侧系统不会在无动作情况下收到任何通知获知到B侧掉线的情况(除非AB是直连状态，那么A可以获知到自己网卡掉线的异常)

在此大前提的基础上，会因为链路环境、SOCKET设定、以及内核相关配置的不同，A侧会在不同的时机获知到B侧无响应的结果，但总归是以异常的形式获得这个结果。

<**关于内核对待无数据传递SOCKET的方式**>

操作系统有一堆时间超级长的兜底用timeout参数，用于在不同的时候给TCP栈一个异常退出的机会，避免无效连接过多而耗尽系统资源

其中，<**TCP KeepAive**>特性能让应用层配置一个远小于内核timeout参数的值，用于在这一堆时间超长的兜底参数生效之前，判断链路是否为有效状态。

<**关于超时的各个节点**>

**以下仅讨论三次握手成功之后的兜底情况**

TCP链路在建立之后，内核会初始化一个由<**nf_conntrack_tcp_timeout_established**>参数控制的计时器（这个计时器在Ubuntu 18.04里面长达5天），以防止在未开启TCP KeepAlive的情况下连接因各种原因导致的长时间无动作而过度消耗系统资源，这个计时器会在每次TCP链路活动后重置

TCP正常传输过程中，每一次数据发送之后，必然伴随对端的ACK确认信息。如果对端因为各种原因失去反应（网络链路中断、意外掉电等）这个ACK将永远不会到来，内核在每次发送之后都会重置一个由<**nf_conntrack_tcp_timeout_unacknowledged**>参数控制的计时器，以防止对端以外断网导致的资源过度消耗。（这个计时器在Ubuntu 18.04里面是300秒/5分钟）

以上两个计时器作为keepalive参数未指定情况下的兜底参数，为内核自保特性，所以事件都很长，建议实际开发与运维中用更为合理的参数覆盖这些数值

<**关于链路异常后发生的操作**>

A侧在超时退出之后一般会发送一个RST包用于告知对端重置链路，并给应用层一个异常的状态信息，视乎同步IO与异步IO的差异，这个异常获知的时机会有所不同。

B侧重启之后，因为不存有之前A-B之间建立链路相关的信息，这时候收到任何A侧来的数据都会以RST作为响应，以告知A侧链路发生异常

**RST的设计用意在于链路发生意料之外的故障时告知链路上的各方释放资源（一般指的是NAT网关与收发两端）;FIN的设计是用于在链路正常情况下的正常单向终止与结束。二者不可混淆。**

<**关于阻塞**>

应用层到底层网卡发送的过程中，数据包会经历多个缓冲区，也会经历一到多次的分片操作，阻塞这一结果的发生是具有从底向上传递的特性。

这一过程中有一个需要强调的关键点：**socket.send**这个操作只是把数据发送到了内核缓冲区，只要数据量不大那么这个调用必然是在拷贝完之后立即返回的。而数据量大的时候，必然会产生阻塞。

在TCP传输中，决定阻塞与否的最终节点，是TCP的可靠传输特性。此特性决定了必须要有ACK数据包回复响应正确接收的数据段范围，内核才会把对应的数据从TCP发送缓冲区中移除，腾出空间让新的数据可以写入进来。

这个过程意味着，只要应用层发送了大于内核缓冲区可容容纳的数据量，那么必然会在应用层出现阻塞，等待ACK的到来，然后把新数据压入缓冲队列，循环往复，直到数据发送完毕。