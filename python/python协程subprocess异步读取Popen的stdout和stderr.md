# python协程subprocess异步读取Popen的stdout和stderr

​		Python的subprocess模块可以很容易地生成子进程，类似Linux系统调用fork和exec。subprocess模块的Popen对象可能以非阻塞的方式调用外部可执行程序，因此我们使用Poen对象来实现需求。如果我们想要把数据写入子进程的标准输入stdin，那么在创建Popen对象的时候就需要指定参数stdin为subprocess.PIPE；同样，如果我们需要从子进程的标准输出中读取数据，那么在创建Popen对象的时候就需要指定参数stdout为subprocess.PIPE。先看一个简单的例子：

```
from subprocess import Popen, PIPE

p = Popen('less', stdin=PIPE, stdout=PIPE)
p.communicate('Line number %d.\n' % x)
```

​		communicate函数返回一个二元组(stdoutdata, stderrdata)，包含了子进程的标准输出和标出错误的输出数据。然而，由于Popen对象的communicate函数会阻塞父进程，同时还会关闭管道，因此每个Popen对象只能调用一次communicate函数，如果有多个请求必须重新生成Popen对象（重新初始化子进程），不能满足我们的需求。

​		因此，我们只有往Popen对象的stdin和stdout对象里写入和读取数据才能实现我们的需求。然而，不幸的是subprocess模块默认情况下只运行在子进程结束的时候读取一次标准输出。Both subprocess and os.popen* only allow input and output one time, and the output to be read only when the process terminates. 



​		通过fcntl模块的fcntl函数可以把子进程的标准输出改为非阻塞的方式，从而达到我们的目的。这样困扰我许久的问题终于得到了完美解决。代码如下：

```
#!/usr/bin/python                                                                                                                                                      
# -*- coding: utf-8 -*-
# author: 
from subprocess import Popen, PIPE
import select
import fcntl, os
import time

class Server(object):
  def __init__(self, args, server_env = None):
    if server_env:
      self.process = Popen(args, stdin=PIPE, stdout=PIPE, stderr=PIPE, env=server_env)
    else:
      self.process = Popen(args, stdin=PIPE, stdout=PIPE, stderr=PIPE)
    flags = fcntl.fcntl(self.process.stdout, fcntl.F_GETFL)
    fcntl.fcntl(self.process.stdout, fcntl.F_SETFL, flags | os.O_NONBLOCK)

  def send(self, data, tail = '\n'):
    self.process.stdin.write(data + tail)
    self.process.stdin.flush()

  def recv(self, t=.1, stderr=0):
    r = ''
    pr = self.process.stdout
    if stderr:
      pr = self.process.stdout
    while True:
      if not select.select([pr], [], [], 0)[0]:
        time.sleep(t)
        continue
      r = pr.read()
      return r.rstrip()
    return r.rstrip()

if __name__ == "__main__":
  ServerArgs = ['/path/to/server', '/path/to/args']
  server = Server(ServerArgs)
  test_data = '拿铁', '咖啡'
  for x in test_data:
    server.send(x)
    print x, server.recv()
```

