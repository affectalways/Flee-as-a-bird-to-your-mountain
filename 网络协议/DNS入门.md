# DNS入门

### 目录

- [DNS 是什么](#DNS是什么)
- [DNS服务器](#DNS服务器)
- [其他DNS工具](#其他DNS工具)
- [参考链接](#参考链接)



</br></br>





DNS 是互联网核心协议之一。不管是上网浏览，还是编程开发，都需要了解一点它的知识。

本文详细介绍DNS的原理，以及如何运用工具软件观察它的运作。我的目标是，读完此文后，你就能完全理解DNS。

![img](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016061513.png)





### DNS是什么

DNS （Domain Name System 的缩写）的作用非常简单，就是根据域名查出IP地址。你可以把它想象成一本巨大的电话本。

举例来说，如果你要访问域名`math.stackexchange.com`，首先要通过DNS查出它的IP地址是`151.101.129.69`。

如果你不清楚为什么一定要查出IP地址，才能进行网络通信，建议先阅读我写的[《互联网协议入门》](https://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html)。



</br></br>

### DNS服务器

下面我们根据例子，一步步还原，本机到底怎么得到域名`math.stackexchange.com`的IP地址。

首先，本机一定要知道DNS服务器的IP地址，否则上不了网。通过DNS服务器，才能知道某个域名的IP地址到底是什么。

![img](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016061507.jpg)

DNS服务器的IP地址，有可能是动态的，每次上网时由网关分配，这叫做DHCP机制；也有可能是事先指定的固定地址。Linux系统里面，DNS服务器的IP地址保存在`/etc/resolv.conf`文件。



</br></br>

### 域名的层级

- ##### 根域名

- ##### 顶级域名

- ##### 域名DNS服务器

DNS服务器怎么会知道每个域名的IP地址呢？答案是分级查询。

#### 根域名

请仔细看前面的例子，每个域名的尾部都多了一个点。

![img](https://www.ruanyifeng.com/blogimg/asset/2016/bg2016061503.png)

比如，域名`math.stackexchange.com`显示为`math.stackexchange.com.`。这不是疏忽，而是所有域名的尾部，实际上都有一个**根域名**。

举例来说，`www.example.com`真正的域名是`www.example.com.root`，简写为`www.example.com.`。因为，根域名`.root`对于所有域名都是一样的，所以平时是省略的。

#### 顶级域名

根域名的下一级，叫做**"顶级域名"（top-level domain，缩写为TLD）**，比如`.com`、`.net`

#### 域名DNS服务器

再下一级叫做**域名DNS服务器**，比如`www.example.com`里面的`.example`，这一级域名是用户可以注册的；

> 主机名

再下一级是**主机名（host）**，比如`www.example.com`里面的`www`，又称为"三级域名"，这是用户在自己的域里面为服务器分配的名称，是用户可以任意分配的。

总结一下，域名的层级结构如下。

> ```bash
> 主机名.域名DNS服务器.顶级域名.根域名
> 
> # 即
> 
> host.sld.tld.root
> ```





</br></br>

### 参考链接

- https://www.ruanyifeng.com/blog/2016/06/dns.html