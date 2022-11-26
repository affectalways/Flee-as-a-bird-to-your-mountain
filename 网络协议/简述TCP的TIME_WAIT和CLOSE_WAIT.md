### 简述 TCP 的 TIME_WAIT 和 CLOSE_WAIT

![](https://segmentfault.com/img/remote/1460000039165595)

# TIME_WAIT

#### 产生

首先调用close()发起主动关闭的一方，在发送最后一个ACK之后会进入TIME_WAIT的状态，也就说该发送方会保持2MSL时间之后才会回到CLOSED状态，**TIME_WAIT只有在先发起主动关闭的一方才会出现。**



#### 作用

- 防止最后的ACK丢失，实现TCP全双工连接的可靠释放。
- 2MSL等待周期可以使旧的数据包在网络过期而消失。

大量的 TIME_WAIT 状态 TCP 连接存在，会占用端口资源，导致无法建立新的连接（无可用端口），其本质原因是什么?

- 大量的短连接存在



#### 如何避免

- 允许TIME_WAIT状态的端口重用；
- 使用长连接





## CLOSE_WAIT

#### 产生

当主动关闭一方A调用close()时，发送接释放报文, 另一方B收到该报文后，会回复ACK，并进入CLOSE_WAIT状态。该状态下， A发送数据给B的链路已经断掉了，但B发送给A的链路并未断开，仍能发送数据。当B调用了close()时，才会退出该状态，进入LAST_ACK的状态。即只有被动关闭的一方才会出现CLOSE_WAIT的状态。



#### 大量异常

CLOSE_WAIT 状态在服务器停留时间很短，如果你发现大量的 CLOSE_WAIT 状态，那么就意味着被动关闭的一方没有及时发出 FIN 包，一般有如下几种可能：

程序问题：即代码层面没有及时 close 相应的 socket 连接，那么自然不会发出 FIN 包，从而导致 CLOSE_WAIT 累积；


### 解决思路

- 分析代码，定位解决。
- 心跳检测：清理无效链接，可以通过应用层来实现，也可以利用TCP的keepalive机制。



#### KEEPALIVE

net.ipv4.tcp_keepalive_time = 7200 (秒)
net.ipv4.tcp_keepalive_intvl = 75 （秒）
net.ipv4.tcp_keepalive_probes = 9 （重复次数）

所以keepalive的超时时长为：

```
keepalive_time + keepalive_probes * keepalive_intv1
```