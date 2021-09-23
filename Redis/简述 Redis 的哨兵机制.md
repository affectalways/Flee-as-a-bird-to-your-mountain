### 简述 Redis 的哨兵机制

### 一、前言

我们在上一章节中讲解了 Redis 主从复制，其实 Redis 主从复制有一个很大的缺点就是没有办法对 master 进行动态选举（当 master 挂掉后，会通过一定的机制，从 slave 中选举出一个新的 master），需要使用 Sentinel 机制完成动态选举，这就是本章节的主题，我们会讲解关于 Redis Sentinel 哨兵机制的相关知识点，我还是使用两台虚拟机进行模拟，**主服务器 ip 为 192.168.1.216，从服务器 ip 为 192.168.1.217（后面就以主从节点命名）**。

------



### **二、Redis Sentinel 哨兵机制介绍**

**1、哨兵机制简介**

1）Sentinel(哨兵) 进程是用于**监控 Redis 集群中 Master 主服务器工作的状态**

2）在 Master 主服务器发生故障的时候，可以实现 Master 和 Slave 服务器的切换，保证系统的高可用（High Availability）

3）哨兵机制被集成在 Redis2.6+ 的版本中，到了2.8版本后就稳定下来了。



**2、哨兵进程的作用**

1）监控(Monitoring)：哨兵(sentinel) 会不断地检查你的 Master 和 Slave 是否运作正常。

2）提醒(Notification)：当被监控的某个Redis节点出现问题时, 哨兵(sentinel) 可以通过 API 向管理员或者其他应用程序发送通知。（使用较少）

3.）自动故障迁移(Automatic failover)：当一个 Master 不能正常工作时，哨兵(sentinel) 会开始一次自动故障迁移操作。具体操作如下：

（1）它会将失效 Master 的其中一个 Slave 升级为新的 Master, 并让失效 Master 的其他Slave 改为复制新的 Master。

（2）当客户端试图连接失效的 Master 时，集群也会向客户端返回新 Master 的地址，使得集群可以使用现在的 Master 替换失效 Master。

（3）**Master 和 Slave 服务器切换后，Master 的 redis.conf、Slave 的 redis.conf 和sentinel.conf 的配置文件的内容都会发生相应的改变，即 Master 主服务器的 redis.conf 配置文件中会多一行 slaveof 的配置，sentinel.conf 的监控目标会随之调换。**



**3、哨兵进程的工作方式**

1）每个 Sentinel（哨兵）进程以**每秒钟一次**的频率向整个集群中的 **Master 主服务器，Slave 从服务器以及其他 Sentinel（哨兵）进程**发送一个 PING 命令。（此处我们还没有讲到集群，下一章节就会讲到，这一点并不影响我们模拟哨兵机制）

\2. 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为**主观下线**（**SDOWN**）。

\3. 如果一个 Master 主服务器被标记为主观下线（SDOWN），则正在监视这个 Master 主服务器的**所有 Sentinel（哨兵）**进程要以每秒一次的频率**确认 Master 主服务器**的确**进入了主观下线状态**。

\4. 当**有足够数量的 Sentinel（哨兵）**进程（大于等于配置文件指定的值）在指定的时间范围内确认 Master 主服务器进入了主观下线状态（SDOWN）， 则 Master 主服务器会被标记为**客观下线（ODOWN）**。

\5. 在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有Master 主服务器、Slave 从服务器发送 INFO 命令。

\6. 当 Master 主服务器被 Sentinel（哨兵）进程标记为**客观下线（ODOWN）**时，Sentinel（哨兵）进程向下线的 Master 主服务器的所有 Slave 从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。

\7. 若没有足够数量的 Sentinel（哨兵）进程同意 Master 主服务器下线， Master 主服务器的客观下线状态就会被移除。若 Master 主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master 主服务器的主观下线状态就会被移除。

![img](https://pic3.zhimg.com/v2-5730dd85717f45863072f36dc89d7b4a_b.jpg)

------

### **三、Redis Sentinel 哨兵机制模拟演示**

1、要想使用 Sentinel 哨兵，我们需要去源码包中拷贝一个配置文件到 Redis/bin 目录（主从节点都可以）下，这个文件就是 **sentinel.conf**，哨兵的配置十分简单，只需要修改这个配置文件中的一个配置即可，这里我们用 vim 命令打开 sentinel.conf 文件，找到文件的第 98 行，修改为 master 的相关信息，mymaster 参数为主节点的名称，可任意修改，而最后一个参数 1，指的是可以参与选举的哨兵的个数，这里我们就使用了一个哨兵，所以值为 1。

![img](https://pic3.zhimg.com/v2-664485577911a5369ace401e9417ca4e_b.jpg)

2、配置完成后，先启动主从节点的 Redis 服务并连接上命令行客户端，然后通过下面的命令启动哨兵服务。

> ./redis-sentinel sentinel.conf

![img](https://pic1.zhimg.com/v2-2982beeafb87635b14c58d7ff7150b20_b.jpg)

![img](https://pic2.zhimg.com/v2-f52df3d8ba2f0f0d1033e40780564f5d_b.jpg)

当出现上面这个界面，说明 Redis 哨兵服务启动成功。 

3、我们通过 info replication 命令来查看一下主从节点的相关信息

1）主节点信息

![img](https://pic1.zhimg.com/v2-332cfac9996d14945c6ca2f43ce87e8c_b.jpg)

2）从节点信息

![img](https://pic3.zhimg.com/v2-419153750ade2c6868b2f7e2cdb3b536_b.jpg)

4、所有的配置都完成了，下面我们来模拟一下主节点宕机，哨兵是如何工作的，我们直接杀死主节点进程：

![img](https://pic4.zhimg.com/v2-3300753f28b4e38cc99a1be50d4fecfb_b.jpg)

杀死进程之后，去观察哨兵给出的响应，这个需要一定的时间

![img](https://pic4.zhimg.com/v2-0020f801f54d79bc6b9061b3043cc707_b.jpg)

从哨兵反馈的日志可以看出，主节点先被 主观下线（SDOWN），经过哨兵选举后状态修改为客观下线（ODOWN），可以看到 ODOWN 命令后面有一个 #quorum（选举人数）1/1，说明哨兵进程大于等于配置文件指定的值，同意客观下线，最后将 192.168.1.217 选举为了主节点，我们使用 info 命令去看一下：（**此后 192.168.1.217 为主节点，192.168.1.216 为从节点**）

![img](https://pic4.zhimg.com/v2-e3c46f0a352a78bea09a7b4b72303b03_b.jpg)

此时，192.168.1.217 已经是主节点了，但是由于 192.168.1.216 的 Redis 服务已经被杀死，所以没有当做从节点连接到 192.168.1.217 节点上，这里我们重新启动 192.168.1.216 节点的 Redis 服务，然后再去看一下两台机器的 info 信息

1）192.168.1.217 主节点

![img](https://pic3.zhimg.com/v2-96e3d0fca2f8e78ad79d6a458d9ebe8a_b.jpg)

2）192.168.1.216 从节点

![img](https://pic1.zhimg.com/v2-9bf3014a1b221be8a5b7e149aa840a68_b.jpg)

5、至此一个简单的哨兵机制就模拟完成了，由于操作截图较为复杂，就不再进行其他的演示了，读者可自己多配置几台虚拟机和哨兵测试一下。

