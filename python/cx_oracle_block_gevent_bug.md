---
title: "Bug: Python第三方库cx_Oracle阻塞gevent"
date: 2020-11-12T21:48:10+08:00
tags: ["python"]
keywords: 
- python
- blog
- 博客
- cx_Oracle
- block gevent
- celery
description: "Python第三方库cx_Oracle 阻塞 gevent"
categories: ["python", "celery"]
draft: false
---

## Bug：Python第三方库cx_Oracle阻塞gevent

#### 问题描述：

线上维护一个项目，flask + celery + gevent + sqlite，下发一个B段弱密码扫描任务，任务跑了一天没结束。



#### 解决流程：

查看celery worker日志发现，三个worker卡在了某个地方。

执行`ps -aux | grep worker`，获取worker的pid信息，因为总共有三个worker，挨个用以下命令查看

`strace -p pid`，举例如下（非实际环境）：

```
$ strace -p 10268 -s 10000
Process 10268 attached - interrupt to quit
recvfrom(5,
```

`10268`是celery worker的`pid`，`recvfrom(5`意味着这个worker在等着从这个文件描述符接收数据

然后通过执行`lsof`命令查看`5`对这个worker进程意味着什么

```py
lsof -p 10268
COMMAND   PID USER   FD   TYPE    DEVICE SIZE/OFF      NODE NAME
......
celery  10268 root    5u  IPv4 828871825      0t0       TCP 127.0.0.1:36162->192.168.18.18:1521 (ESTABLISHED)
```

上面的信息表示worker卡在了tcp connection上。

又因为`1521`端口是oracle数据库的默认端口，然后查看该服务器上是否运行oracle服务，发现确实是的。

所以排查范围进一步缩小，定位到了是oracle弱密码爆破插件的问题。

继续进行下一步定位，通过查看源码发现，代码没问题，只有引入的第三方库`cx_Oracle`不确定，所以认为是该库导致的问题。

想看下`cx_Oracle`源码，发现mmp的不开源。

只能谷歌搜索`"cx_Oracle block gevent"`，发现还真有遇到的。网址如下：<https://blog.csdn.net/chuzhuanbo7093/article/details/100983954>

```
cx_Oracle的OCI不支持异步，导致gevent在协程处理请求时，如果数据库长时间不返回，系统就会被阻塞，从而使后续请求得不到处理，最终系统崩溃。
关于cx_Oracle的OCI不支持异步，请参考：
https://bitbucket.org/anthony_tuininga/cx_oracle/issues/8/gevent-eventlet-support
```

**问题定位到了，确认是cx_Oracle导致**。但是cx_oracle代码**不开源**，不能patch或给个hook。



因为不能patch，所以只能采用其他解决方案对`cx_Oracle`连接时间进行限制。

两种方案：

​	1.用eventlet对oracle连接进行时间限制（这是**因为能阻塞gevent的，只要用eventlet替换gevent就可以了**，但是参考连接忘了保存了，希望有人能补上！！！）	

​	2.用thread对oracle连接进行时间限制

**第一种方案：**

```
import eventlet
eventlet.monkey_patch()
with eventlet.Timeout() as t:
	pass
```

把弱密码代码写成如上方式对连接时间进行限制，单独运行是可以的。**但是**，一放到环境中运行就报错了，报错如下：

```
>>> import eventlet
>>> 
>>> eventlet.monkey_patch()
>>> import select
>>> select.poll
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: module 'select' has no attribute 'poll'
```

这是因为`eventlet`打了patch后，会把poll属性删除，[参考如下](https://stackoverflow.com/questions/61152057/eventlet-breaks-select-poll)

```
It's intentional, as select.poll() is not "green".

See also: https://github.com/eventlet/eventlet/issues/608#issuecomment-612359458
```

[源代码连接](https://github.com/eventlet/eventlet/commit/f63165c0e3c85699ebdb454878d1eaea13e90553)：<https://github.com/eventlet/eventlet/commit/f63165c0e3c85699ebdb454878d1eaea13e90553>

```
eventlet/green/select.py


__patched__ = ['select']	__patched__ = ['select']
__deleted__ = ['poll', 'epoll', 'kqueue', 'kevent']




def get_fileno(obj):	def get_fileno(obj):
```

在尝试了**不进行monkey_patch**、**不对select patch**、**手动添加poll**属性，均不行。

其中有个报错是：

```
TypeError: Cannot convert greenlet.greenlet to gevent.__greenlet_primitives.SwitchOutGreenletWithLo
```

对应解决方案：<https://stackoverflow.com/questions/58191609/gevent-throws-exception-from-wrong-thread-greenlet>

完全解决不了，所以就放弃第一种方案了。



采用第二种方案：

还别说，换上就可以了，除了报了一个很小的错误：

```
Cannot route message for exchange 'reply.celery.pidbox': Table empty or key no longer exists.
Probably the key ('_kombu.binding.reply.celery.pidbox') has been removed from the Redis database.: 
```

解决这个小bug的方案就是：**降低celery的依赖kombu版本**，连接：<https://stackoverflow.com/questions/58896295/cannot-route-message-for-exchange-reply-celery-pidbox-table-empty-or-key-no-l>

<https://github.com/celery/kombu/issues/1063>





#### 参考链接：

https://blog.csdn.net/chuzhuanbo7093/article/details/100983954

<https://stackoverflow.com/questions/61152057/eventlet-breaks-select-poll>

<https://stackoverflow.com/questions/30272845/celery-worker-hangs-without-any-error>

<https://www.caktusgroup.com/blog/2013/10/30/using-strace-debug-stuck-celery-tasks/>

https://stackoverflow.com/questions/61152057/eventlet-breaks-select-poll

https://github.com/eventlet/eventlet/issues/608#issuecomment-612359458

https://github.com/eventlet/eventlet/commit/f63165c0e3c85699ebdb454878d1eaea13e90553

<https://github.com/celery/kombu/issues/1063>

<https://stackoverflow.com/questions/58896295/cannot-route-message-for-exchange-reply-celery-pidbox-table-empty-or-key-no-l>