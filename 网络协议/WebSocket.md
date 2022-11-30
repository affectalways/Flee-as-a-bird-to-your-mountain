# WebSocket

### 目录

- [是什么](#是什么)
- [Websocket握手过程](#Websocket握手过程)
- [WebSocket与Http的区别](#WebSocket与Http的区别)
- [为什么需要Websocket](#为什么需要Websocket)
- [WebSocket与Http的区别](#WebSocket与Http的区别)
- [参考链接](#参考链接)



### 是什么

> WebSocket是应用层协议

WebSocket是**基于TCP的应用层协议**，用于在C/S架构的应用中实现**双向通信**，关于WebSocket协议的详细规范和定义参见[rfc6455](https://tools.ietf.org/html/rfc6455)。

它的**最大特点**就是，**服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。**

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/websocket2.png)

需要特别注意的是：虽然WebSocket协议在建立连接时会使用HTTP协议，但这并不意味着WebSocket协议是基于HTTP协议实现的，Websocket其实是一个新协议，跟HTTP协议基本没有关系，只是为了兼容现有浏览器的握手规范而已，也就是说它是HTTP协议上的一种补充可以通过这样一张图理解

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/websocket1.png)

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

> ```markup
> ws://example.com:80/some/path
> ```

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/websocket3.png)



> 虽然WebSocket和HTTP是不同应用协议，但[rfc6455](https://tools.ietf.org/html/rfc6455)规定：“WebSocket设计为通过80和443端口工作，以及支持HTTP代理和中介”，从而使其与HTTP协议兼容。为了实现兼容性，WebSocket握手时使用HTTP Upgrade头从HTTP协议更改为WebSocket协议，参考：[WebSocket维基百科](https://zh.wikipedia.org/wiki/WebSocket) 。



</br></br>

### Websocket握手过程

> - 首先，客户端发起http请求，经过3次握手后，建立起TCP连接；http请求里存放WebSocket支持的版本号等信息，如：Upgrade、Connection、WebSocket-Version等；
> - 然后，服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据；
> - 最后，客户端收到连接成功的消息后，开始借助于TCP传输信道进行全双工通信。

出于兼容性的考虑，WS 的握手使用 HTTP 来实现，客户端的握手消息就是一个「普通的，带有 Upgrade 头的，HTTP Request 消息」。所以这一个小节到内容大部分都来自于 RFC2616，这里只是它的一种应用形式，下面是 RFC6455 文档中给出的一个客户端握手消息示例：

```js
GET /chat HTTP/1.1            //1
Host: server.example.com   //2

Upgrade: websocket            //3
Connection: Upgrade            //4

Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==            //5
Origin: http://example.com            //6
Sec-WebSocket-Protocol: chat, superchat            //7
Sec-WebSocket-Version: 13            //8
```

**这段类似HTTP协议的握手请求中，多了几个东西**

```
Upgrade: websocket
Connection: Upgrade
```

**这个就是Websocket的核心，告诉服务器：发起的是Websocket协议**

可以看到，前两行跟 HTTP 的 Request 的起始行一模一样，而真正在 WS 的握手过程中起到作用的是下面几个 header。

1. Upgrade：upgrade 是 HTTP1.1 中用于定义转换协议的 header 域。它表示，如果服务器支持的话，客户端希望使用现有的「网络层」已经建立好的这个「连接（此处是 TCP 连接）」，切换到另外一个「应用层」（此处是 WebSocket）协议。
2. Connection：HTTP1.1 中规定 Upgrade 只能应用在「直接连接」中，所以带有 Upgrade 头的 HTTP1.1 消息必须含有 Connection 头，因为 Connection 头的意义就是，任何接收到此消息的人（往往是代理服务器）都要在转发此消息之前处理掉 Connection 中指定的域（不转发 Upgrade 域）。

</br>

如果客户端和服务器之间是通过代理连接的，那么在发送这个握手消息之前首先要发送 CONNECT 消息来建立直接连接。

1. Sec-WebSocket-＊：第 7 行标识了客户端支持的子协议的列表（关于子协议会在下面介绍），第 8 行标识了客户端支持的 WS 协议的版本列表，第 5 行用来发送给服务器使用（服务器会使用此字段组装成另一个 key 值放在握手返回信息里发送客户端）。
2. Origin：作安全使用，防止跨站攻击，浏览器一般会使用这个来标识原始域。

如果服务器接受了这个请求，可能会发送如下这样的返回信息，这是一个标准的 HTTP 的 Response 消息。101 表示服务器收到了客户端切换协议的请求，并且同意切换到此协议。RFC2616 规定只有 HTTP1.1 及 HHTTP1.1 以上版本的时候才能同意切换。

```html
HTTP/1.1 101 Switching Protocols //1
Upgrade: websocket. //2
Connection: Upgrade. //3
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  //4
Sec-WebSocket-Protocol: chat. //5
```

**ws 协议默认使用 80 端口，wss 协议默认使用 443 端口。（和 http 一样啊:relaxed:）**



</br></br>

### WebSocket与Http的区别

- 通信方式不同

WebSocket是**双向通信**模式，客户端与服务器之间只有在握手阶段是使用HTTP协议的“请求-响应”模式交互，而一旦连接建立之后的通信则使用**双向模式**交互，不论是客户端还是服务端都可以随时将数据发送给对方

而HTTP协议则至始至终都采用**“请求-响应”模式**进行通信。也正因为如此，HTTP协议的通信效率没有WebSocket高。

> 在HTTP中永远是这样，也就是说一个request只能有一个response。而且这个response也是**被动**的，不能主动发起。

##### ![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/websocket2.png)

- ##### 协议格式不同

WebSocket与HTTP的协议格式是完全不同的，具体来讲：
（1）HTTP协议（参见：[rfc2616](https://tools.ietf.org/html/rfc2616)）比较臃肿，而WebSocket协议比较轻量。
（2）对于HTTP协议来讲，一个数据包就是一条完整的消息；而WebSocket客户端与服务端通信的最小单位是帧（frame），由1个或多个帧组成一条完整的消息（message）。即：发送端将消息切割成多个帧，并发送给服务端；服务端接收消息帧，并将关联的帧重新组装成完整的消息。



</br></br>

### 为什么需要Websocket

随着Web应用的发展，特别是动态网页的普及，越来越多的场景需要实现数据动态刷新。
在早期的时候，实现数据刷新的方式通常有如下3种：

- #### 客户端定时查询

客户端定时查询（如：每隔10秒钟查询一次）是最原始也是最简单的实现数据刷新的方法，服务端不用做任何改动，只需要在客户端添加一个定时器即可。但是这种方式的缺点也很明显：大量的定时请求都是无效的，因为服务端的数据并没有更新，相应地也导致了大量的带宽浪费。

- #### 长轮训机制

长轮训机制是对客户端定时查询的一种改进，即：客户端依旧保持定时发送请求给服务端，但是服务端并不立即响应，而是等到真正有数据更新的时候才发送给客户端。实际上，并不是当没有数据更新时服务端就永远都不响应客户端，而是需要在等待一个超时时间之后结束该次长轮训请求。相对于客户端定时查询方式而言，当数据更新频率不确定时长轮训机制能够很明显地减少请求数。但是，在数据更新比较频繁的场景下，长轮训方式的优势就没那么明显了。

在Web开发中使用得最为普遍的长轮训实现方案为Comet（[Comet (web技术)](https://zh.wikipedia.org/wiki/Comet_(web技术))），Tomcat和Jetty都有对应的实现支持，详见：[WhatIsComet](https://wiki.apache.org/tomcat/WhatIsComet)，[Why Asynchronous Servlets](https://www.eclipse.org/jetty/documentation/9.4.x/continuations.html#continuations-intro)。

- #### HTTP Streaming

不论是长轮训机制还是传统的客户端定时查询方式，都需要客户端不断地发送请求以获取数据更新，而HTTP Streaming则试图改变这种方式，其实现机制为：客户端发送获取数据更新请求到服务端时，服务端将保持该请求的响应数据流一直打开，只要有数据更新就实时地发送给客户端。

虽然这个设想是非常美好的，但这带来了新的问题：

(1)HTTP Streaming的实现机制违背了HTTP协议本身的语义，使得客户端与服务端不再是“请求-响应”的交互方式，而是直接在二者建立起了一个单向的“通信管道”。

(2)在HTTP Streaming模式下，服务端只要得到数据更新就发送给客户端，那么就需要客户端与服务端协商如何区分每一个更新数据包的开始和结尾，否则就可能出现解析数据错误的情况。

(3)另外，处于客户端与服务端的网络中介（如：代理）可能会缓存响应数据流，这可能会导致客户端无法真正获取到服务端的更新数据，这实际上与HTTP Streaming的本意是相违背的。

鉴于上述原因，在实际应用中HTTP Streaming并没有真正流行起来，反之使用得最多的是长轮训机制。

显然，上述几种实现数据动态刷新的方式都是基于HTTP协议实现的，或多或少地存在这样那样的问题和缺陷；而**WebSocket**是一个全新的应用层协议，专门用于Web应用中需要实现动态刷新的场景。

相比起HTTP协议，WebSocket具备如下特点：

1. 支持双向通信，实时性更强。
2. 更好的二进制支持。
3. 较少的控制开销：连接创建后，WebSockete客户端、服务端进行数据交换时，协议控制的数据包头部较小。
4. 支持扩展。



</br></br>

### 参考链接

- https://www.ruanyifeng.com/blog/2017/05/websocket.html
- https://www.cnblogs.com/chyingp/p/websocket-deep-in.html
- https://www.cnblogs.com/nuccch/p/10947256.html
- https://cloud.tencent.com/developer/article/1005043