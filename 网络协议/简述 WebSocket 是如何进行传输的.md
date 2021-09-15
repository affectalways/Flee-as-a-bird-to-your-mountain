# WebSocket 详解



## WebSocket 出现前

构建网络应用的过程中，我们经常需要与服务器进行持续的通讯以保持双方信息的同步。通常这种持久通讯在不刷新页面的情况下进行，消耗一定的内存资源常驻后台，并且对于用户不可见。在 WebSocket 出现之前，我们有一下解决方案：



### 传统轮询(Traditional Polling)

当前Web应用中较常见的一种持续通信方式，通常采取 `setInterval` 或者 `setTimeout` 实现。例如如果我们想要定时获取并刷新页面上的数据，可以结合Ajax写出如下实现：

```
setInterval(function() {
    $.get("/path/to/server", function(data, status) {
        console.log(data);
    });
}, 10000);
```

上面的程序会每隔10秒向服务器请求一次数据，并在数据到达后存储。这个实现方法通常可以满足简单的需求，然而同时也存在着很大的缺陷：在网络情况不稳定的情况下，服务器从接收请求、发送请求到客户端接收请求的总时间有可能超过10秒，而请求是以10秒间隔发送的，这样会导致接收的数据到达先后顺序与发送顺序不一致。于是出现了采用 `setTimeout` 的轮询方式：

```
function poll() {
    setTimeout(function() {
        $.get("/path/to/server", function(data, status) {
            console.log(data);
            // 发起下一次请求
            poll();
        });
    }, 10000);
}
```

程序首先设置10秒后发起请求，当数据返回后再隔10秒发起第二次请求，以此类推。这样的话虽然无法保证两次请求之间的时间间隔为固定值，但是可以保证到达数据的顺序。



### 长轮询(Long Polling)

上面两种传统的轮询方式都存在一个严重缺陷：程序在每次请求时都会新建一个HTTP请求，然而并不是每次都能返回所需的新数据。当同时发起的请求达到一定数目时，会对服务器造成较大负担。这时我们可以采用长轮询方式解决这个问题。

> 长轮询与以下将要提到的服务器发送事件和WebSocket不能仅仅依靠客户端JavaScript实现，我们同时需要服务器支持并实现相应的技术。

长轮询的基本思想是在每次客户端发出请求后，服务器检查上次返回的数据与此次请求时的数据之间是否有更新，如果有更新则返回新数据并结束此次连接，否则服务器 hold`住此次连接，直到有新数据时再返回相应。而这种长时间的保持连接可以通过设置一个较大的`HTTP timeout` 实现。下面是一个简单的长连接示例：

服务器（PHP）：

```
<?php
    // 示例数据为data.txt
    $filename= dirname(__FILE__)."/data.txt";
    // 从请求参数中获取上次请求到的数据的时间戳
    $lastmodif = isset( $_GET["timestamp"])? $_GET["timestamp"]: 0 ;
    // 将文件的最后一次修改时间作为当前数据的时间戳
    $currentmodif = filemtime($filename);

    // 当上次请求到的数据的时间戳*不旧于*当前文件的时间戳，使用循环"hold"住当前连接，并不断获取文件的修改时间
    while ($currentmodif <= $lastmodif) {
        // 每次刷新文件信息的时间间隔为10秒
        usleep(10000);
        // 清除文件信息缓存，保证每次获取的修改时间都是最新的修改时间
        clearstatcache();
        $currentmodif = filemtime($filename);
    }

    // 返回数据和最新的时间戳，结束此次连接
    $response = array();
    $response["msg"] =Date("h:i:s")." ".file_get_contents($filename);
    $response["timestamp"]= $currentmodif;
    echo json_encode($response);
?>
```

客户端：

```
function longPoll (timestamp) {
    var _timestamp;
    $.get("/path/to/server?timestamp=" + timestamp)
    .done(function(res) {
        try {
            var data = JSON.parse(res);
            console.log(data.msg);
            _timestamp = data.timestamp;
        } catch (e) {}
    })
    .always(function() {
        setTimeout(function() {
            longPoll(_timestamp || Date.now()/1000);
        }, 10000);
    });
}
```

长轮询可以有效地解决传统轮询带来的带宽浪费，但是每次连接的保持是以消耗服务器资源为代价的。尤其对于Apache+PHP 服务器，由于有默认的 `worker threads` 数目的限制，当长连接较多时，服务器便无法对新请求进行相应。



### 对比

| >>>>>>>>>>>> | 传统轮询                                | 长轮询                               | WebSocket                                                    |
| ------------ | --------------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| 服务器负载   | 较少的CPU资源，较多的内存资源和带宽资源 | 与传统轮询相似，但是占用带宽较少     | 无需循环等待（长轮询），CPU和内存资源不以客户端数量衡量，而是以客户端事件数衡量。四种方式里性能最佳。 |
| 客户端负载   | 占用较多的内存资源与请求数。            | 与传统轮询相似。                     | 小的很                                                       |
| 延迟         | 非实时，延迟取决于请求间隔。            | 同传统轮询。                         | 实时。                                                       |
| 实现复杂度   | 非常简单。                              | 需要服务器配合，客户端实现非常简单。 | 需要Socket程序实现和额外端口，客户端实现简单。               |





## WebSocket 是什么

WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。

WebSocket同样是HTML 5规范的组成部分之一，现标准版本为 RFC 6455。WebSocket 相较于上述几种连接方式，实现原理较为复杂，用一句话概括就是：客户端向 WebSocket 服务器通知（notify）一个带有所有 `接收者ID（recipients IDs）` 的事件（event），服务器接收后立即通知所有活跃的（active）客户端，只有ID在接收者ID序列中的客户端才会处理这个事件。由于 WebSocket 本身是基于TCP协议的，所以在服务器端我们可以采用构建 TCP Socket 服务器的方式来构建 WebSocket 服务器。

这个 WebSocket 是一种全新的协议。它将 TCP 的 `Socket（套接字）`应用在了web page上，从而使通信双方建立起一个保持在活动状态连接通道，并且属于全双工（双方同时进行双向通信）。

其实是这样的，WebSocket 协议是借用 HTTP协议 的 101 switch protocol 来达到协议转换的，从HTTP协议切换成WebSocket通信协议。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。其他特点包括：

- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有着良好的兼容性。默认端口也是 80 和 443 ，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。



## 协议

WebSocket协议被设计来取代现有的使用HTTP作为传输层的双向通信技术，并受益于现有的基础设施（代理、过滤、身份验证）。



### 概述

本协议有两部分：握手和数据传输。

来自客户端的握手看起来像如下形式：

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

来自服务器的握手看起来像如下形式：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

来自客户端的首行遵照 Request-Line 格式。 来自服务器的首行遵照 Status-Line 格式。

Request-Line 和 Status-Line 制品定义在 [RFC2616](http://tools.ietf.org/html/rfc2616)。

一旦客户端和服务器都发送了它们的握手，且如果握手成功，接着开始数据传输部分。 这是一个每一端都可以的双向通信信道，彼此独立，随意发生数据。

一个成功握手之后，客户端和服务器来回地传输数据，在本规范中提到的概念单位为“消息”。 在线路上，一个消息是由一个或多个帧的组成。 WebSocket 的消息并不一定对应于一个特定的网络层帧，可以作为一个可以被一个中间件合并或分解的片段消息。

一个帧有一个相应的类型。 属于相同消息的每一帧包含相同类型的数据。 从广义上讲，有文本数据类型（它被解释为 UTF-8 [RFC3629](http://tools.ietf.org/html/rfc3629)文本）、二进制数据类型（它的解释是留给应用）、和控制帧类型（它是不准备包含用于应用的数据，而是协议级的信号，例如应关闭连接的信号）。这个版本的协议定义了六个帧类型并保留10以备将来使用。



### 握手

#### 客户端：申请协议升级

首先，客户端发起协议升级请求。可以看到，采用的是标准的 HTTP 报文格式，且只支持GET方法。

```
GET / HTTP/1.1
Host: localhost:8080
Origin: http://127.0.0.1:3000
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==
```

重点请求首部意义如下：

- Connection: Upgrade：表示要升级协议
- Upgrade: websocket：表示要升级到 websocket 协议。
- Sec-WebSocket-Version: 13：表示 websocket 的版本。如果服务端不支持该版本，需要返回一个 `Sec-WebSocket-Versionheader` ，里面包含服务端支持的版本号。
- Sec-WebSocket-Key：与后面服务端响应首部的 `Sec-WebSocket-Accept` 是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。

#### 服务端：响应协议升级

服务端返回内容如下，状态代码101表示协议切换。到此完成协议升级，后续的数据交互都按照新的协议来。

```
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

#### Sec-WebSocket-Accept

`Sec-WebSocket-Accept` 根据客户端请求首部的 `Sec-WebSocket-Key` 计算出来。

计算公式为：

将 `Sec-WebSocket-Key` 跟 `258EAFA5-E914-47DA-95CA-C5AB0DC85B11` 拼接。

通过 SHA1 计算出摘要，并转成 base64 字符串。

伪代码如下：

```
>toBase64( sha1( Sec-WebSocket-Key + 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 )  )
```

### 数据帧

WebSocket 客户端、服务端通信的最小单位是 `帧（frame）`，由 1 个或多个帧组成一条完整的 `消息（message）`。

- 发送端：将消息切割成多个帧，并发送给服务端；
- 接收端：接收消息帧，并将关联的帧重新组装成完整的消息；

#### 数据帧格式概览

用于数据传输部分的报文格式是通过本节中详细描述的 ABNF 来描述。

下面给出了 WebSocket 数据帧的统一格式。熟悉 TCP/IP 协议的同学对这样的图应该不陌生。

从左到右，单位是比特。比如 `FIN`、`RSV1`各占据 1 比特，`opcode`占据 4 比特。

内容包括了标识、操作代码、掩码、数据、数据长度等。

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
```

#### 数据帧格式详解

针对前面的格式概览图，这里逐个字段进行讲解，如有不清楚之处，可参考协议规范，或留言交流。

##### FIN：1 个比特。

如果是 1，表示这是 `消息（message）`的最后一个分片`（fragment）`，如果是 0，表示不是是 `消息（message）`的最后一个 `分片（fragment）`。

##### RSV1, RSV2, RSV3：各占 1 个比特。

一般情况下全为 0。当客户端、服务端协商采用 WebSocket 扩展时，这三个标志位可以非 0，且值的含义由扩展进行定义。如果出现非零的值，且并没有采用 WebSocket 扩展，连接出错。

##### Opcode: 4 个比特。

操作代码，Opcode 的值决定了应该如何解析后续的 `数据载荷（data payload）`。如果操作代码是不认识的，那么接收端应该 `断开连接（fail the connection）`。可选的操作代码如下：

- %x0：表示一个延续帧。当 Opcode 为 0 时，表示本次数据传输采用了数据分片，当前收到的数据帧为其中一个数据分片。
- %x1：表示这是一个文本帧（frame）
- %x2：表示这是一个二进制帧（frame）
- %x3-7：保留的操作代码，用于后续定义的非控制帧。
- %x8：表示连接断开。
- %x8：表示这是一个 ping 操作。
- %xA：表示这是一个 pong 操作。
- %xB-F：保留的操作代码，用于后续定义的控制帧。

##### Mask: 1 个比特。

表示是否要对数据载荷进行掩码操作。**从客户端向服务端发送数据时，需要对数据进行掩码操作；从服务端向客户端发送数据时，不需要对数据进行掩码操作**。

如果服务端接收到的数据没有进行过掩码操作，服务端需要断开连接。

如果 Mask 是 1，那么在 `Masking-key` 中会定义一个 `掩码键（masking key）`，并用这个掩码键来对数据载荷进行反掩码。所有客户端发送到服务端的数据帧，Mask 都是 1。

##### Payload length：数据载荷的长度

单位是字节。为 7 位，或 7+16 位，或 1+64 位。

假设数 Payload length === x，如果

- x 为 0~126：数据的长度为 x 字节。
- x 为 126：后续 2 个字节代表一个 16 位的无符号整数，该无符号整数的值为数据的长度。
- x 为 127：后续 8 个字节代表一个 64 位的无符号整数（最高位为 0），该无符号整数的值为数据的长度。

此外，如果 `payload length` 占用了多个字节的话，`payload length` 的二进制表达采用 `网络序（big endian，重要的位在前）`。

##### Masking-key：0 或 4 字节（32 位）

所有从客户端传送到服务端的数据帧，数据载荷都进行了掩码操作，Mask 为 1，且携带了 4 字节的 `Masking-key`。如果 Mask 为 0，则没有 `Masking-key`。

备注：载荷数据的长度，不包括 mask key 的长度。

##### Payload data：(x+y) 字节

载荷数据：包括了扩展数据、应用数据。其中，扩展数据 x 字节，应用数据 y 字节。

扩展数据：如果没有协商使用扩展的话，扩展数据数据为 0 字节。所有的扩展都必须声明扩展数据的长度，或者可以如何计算出扩展数据的长度。此外，扩展如何使用必须在握手阶段就协商好。如果扩展数据存在，那么载荷数据长度必须将扩展数据的长度包含在内。

应用数据：任意的应用数据，在扩展数据之后（如果存在扩展数据），占据了数据帧剩余的位置。载荷数据长度 减去 扩展数据长度，就得到应用数据的长度。

#### 掩码算法

`掩码键（Masking-key）`是由客户端挑选出来的 32 位的随机数。掩码操作不会影响数据载荷的长度。掩码、反掩码操作都采用如下算法：

首先，假设：

- original-octet-i：为原始数据的第 i 字节。
- transformed-octet-i：为转换后的数据的第 i 字节。
- j：为i mod 4的结果。
- masking-key-octet-j：为 mask key 第 j 字节。

算法描述为： original-octet-i 与 masking-key-octet-j 异或后，得到 transformed-octet-i。

```
j  = i MOD 4
transformed-octet-i = original-octet-i XOR masking-key-octet-j
```

### 数据传递

一旦 WebSocket 客户端、服务端建立连接后，后续的操作都是基于数据帧的传递。

WebSocket 根据 `opcode` 来区分操作的类型。比如0x8表示断开连接，`0x0-0x2` 表示数据交互。

#### 数据分片

WebSocket 的每条消息可能被切分成多个数据帧。当 WebSocket 的接收方收到一个数据帧时，会根据FIN的值来判断，是否已经收到消息的最后一个数据帧。

FIN=1 表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧。

此外，`opcode` 在数据交换的场景下，表示的是数据的类型。0x01表示文本，0x02表示二进制。而0x00比较特殊，表示`延续帧（continuation frame）`，顾名思义，就是完整消息对应的数据帧还没接收完。

### 连接保持 + 心跳

WebSocket 为了保持客户端、服务端的实时双向通信，需要确保客户端、服务端之间的 TCP 通道保持连接没有断开。然而，对于长时间没有数据往来的连接，如果依旧长时间保持着，可能会浪费包括的连接资源。

但不排除有些场景，客户端、服务端虽然长时间没有数据往来，但仍需要保持连接。这个时候，可以采用心跳来实现。

- 发送方 ->接收方：ping
- 接收方 ->发送方：pong

ping、pong 的操作，对应的是 WebSocket 的两个控制帧，opcode分别是 `0x9、0xA`。

### 关闭连接

一旦发送或接收到一个Close控制帧，这就是说，_WebSocket 关闭阶段握手已启动，且 WebSocket 连接处于 CLOSING 状态。

当底层TCP连接已关闭，这就是说 WebSocket连接已关闭 且 WebSocket 连接处于 CLOSED 状态。 如果 TCP 连接在 WebSocket 关闭阶段已经完成后被关闭，WebSocket连接被说成已经 完全地 关闭了。

如果WebSocket连接不能被建立，这就是说，WebSocket连接关闭，但不是 完全的 。

### 状态码

当关闭一个已经建立的连接（例如，当在打开阶段握手已经完成后发送一个关闭帧），端点可以表明关闭的原因。 由端点解释这个原因，并且端点应该给这个原因采取动作，本规范是没有定义的。 本规范定义了一组预定义的状态码，并指定哪些范围可以被扩展、框架和最终应用使用。 状态码和任何相关的文本消息是关闭帧的可选的组件。

当发送关闭帧时端点可以使用如下预定义的状态码。

| 状态码    | 名称                 | 描述                                                         |
| --------- | -------------------- | ------------------------------------------------------------ |
| 0–999     |                      | 保留段, 未使用.                                              |
| 1000      | CLOSE_NORMAL         | 正常关闭; 无论为何目的而创建, 该链接都已成功完成任务.        |
| 1001      | CLOSE_GOING_AWAY     | 终端离开, 可能因为服务端错误, 也可能因为浏览器正从打开连接的页面跳转离开. |
| 1002      | CLOSE_PROTOCOL_ERROR | 由于协议错误而中断连接.                                      |
| 1003      | CLOSE_UNSUPPORTED    | 由于接收到不允许的数据类型而断开连接 (如仅接收文本数据的终端接收到了二进制数据). |
| 1004      |                      | 保留. 其意义可能会在未来定义.                                |
| 1005      | CLOSE_NO_STATUS      | 保留. 表示没有收到预期的状态码.                              |
| 1006      | CLOSE_ABNORMAL       | 保留. 用于期望收到状态码时连接非正常关闭 (也就是说, 没有发送关闭帧). |
| 1007      | Unsupported Data     | 由于收到了格式不符的数据而断开连接 (如文本消息中包含了非 UTF-8 数据). |
| 1008      | Policy Violation     | 由于收到不符合约定的数据而断开连接. 这是一个通用状态码, 用于不适合使用 1003 和 1009 状态码的场景. |
| 1009      | CLOSE_TOO_LARGE      | 由于收到过大的数据帧而断开连接.                              |
| 1010      | Missing Extension    | 客户端期望服务器商定一个或多个拓展, 但服务器没有处理, 因此客户端断开连接. |
| 1011      | Internal Error       | 客户端由于遇到没有预料的情况阻止其完成请求, 因此服务端断开连接. |
| 1012      | Service Restart      | 服务器由于重启而断开连接.                                    |
| 1013      | Try Again Later      | 服务器由于临时原因断开连接, 如服务器过载因此断开一部分客户端连接. |
| 1014      |                      | 由 WebSocket 标准保留以便未来使用.                           |
| 1015      | TLS Handshake        | 保留. 表示连接由于无法完成 TLS 握手而关闭 (例如无法验证服务器证书). |
| 1016–1999 |                      | 由 WebSocket 标准保留以便未来使用.                           |
| 2000–2999 |                      | 由 WebSocket 拓展保留使用.                                   |
| 3000–3999 |                      | 可以由库或框架使用.不应由应用使用. 可以在 IANA 注册, 先到先得. |
| 4000–4999 |                      | 可以由应用使用.                                              |

## 客户端的 API

### WebSocket 构造函数

WebSocket 对象提供了用于创建和管理 WebSocket 连接，以及可以通过该连接发送和接收数据的 API。

WebSocket 构造器方法接受一个必须的参数和一个可选的参数：

```
WebSocket WebSocket(in DOMString url, in optional DOMString protocols);
WebSocket WebSocket(in DOMString url,in optional DOMString[] protocols);
```

### 参数

- url
  表示要连接的URL。这个URL应该为响应WebSocket的地址。
- protocols 可选
  可以是一个单个的协议名字字符串或者包含多个协议名字字符串的数组。这些字符串用来表示子协议，这样做可以让一个服务器实现多种 WebSocket子协议（例如你可能希望通过制定不同的协议来处理不同类型的交互）。如果没有制定这个参数，它会默认设为一个空字符串。

构造器方法可能抛出以下异常：`SECURITY_ERR` 试图连接的端口被屏蔽。

```
var ws = new WebSocket('ws://localhost:8080');
```

执行上面语句之后，客户端就会与服务器进行连接。

### 属性

| 属性名         | 类型           | 描述                                                         |
| -------------- | -------------- | ------------------------------------------------------------ |
| binaryType     | DOMString      | 一个字符串表示被传输二进制的内容的类型。取值应当是"blob"或者"arraybuffer"。"blob"表示使用DOM [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 对象，而"arraybuffer"表示使用 ArrayBuffer 对象。 |
| bufferedAmount | unsigned long  | 调用 [send()](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#send()) 方法将多字节数据加入到队列中等待传输，但是还未发出。该值会在所有队列数据被发送后重置为 0。而当连接关闭时不会设为0。如果持续调用send()，这个值会持续增长。只读。 |
| extensions     | DOMString      | 服务器选定的扩展。目前这个属性只是一个空字符串，或者是一个包含所有扩展的列表。 |
| onclose        | EventListener  | 用于监听连接关闭事件监听器。当 WebSocket 对象的readyState 状态变为 CLOSED 时会触发该事件。这个监听器会接收一个叫close的 [CloseEvent](https://developer.mozilla.org/en/WebSockets/WebSockets_reference/CloseEvent) 对象。 |
| onerror        | EventListener  | 当错误发生时用于监听error事件的事件监听器。会接受一个名为“error”的event对象。 |
| onmessage      | EventListener  | 一个用于消息事件的事件监听器，这一事件当有消息到达的时候该事件会触发。这个Listener会被传入一个名为"message"的 [MessageEvent](https://developer.mozilla.org/en/WebSockets/WebSockets_reference/MessageEvent) 对象。 |
| onopen         | EventListener  | 一个用于连接打开事件的事件监听器。当readyState的值变为 OPEN 的时候会触发该事件。该事件表明这个连接已经准备好接受和发送数据。这个监听器会接受一个名为"open"的事件对象。 |
| protocol       | DOMString      | 一个表明服务器选定的子协议名字的字符串。这个属性的取值会被取值为构造器传入的protocols参数。 |
| readyState     | unsigned short | 连接的当前状态。取值是 [Ready state constants](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#Ready_state_constants) 之一。 只读。 |
| url            | DOMString      | 传入构造器的URL。它必须是一个绝对地址的URL。只读。           |

#### webSocket.onopen

实例对象的 onopen 属性，用于指定连接成功后的回调函数。

```
ws.onopen = function () {
  ws.send('Hello Server!');
}
```

如果要指定多个回调函数，可以使用addEventListener方法。

```
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```

#### webSocket.onclose

实例对象的 onclose 属性，用于指定连接关闭后的回调函数。

```
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};

ws.addEventListener("close", function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
});
```

#### webSocket.onmessage

实例对象的 onmessage 属性，用于指定收到服务器数据后的回调函数。

```
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};

ws.addEventListener("message", function(event) {
  var data = event.data;
  // 处理数据
});
```

注意，服务器数据可能是文本，也可能是 二进制数据（blob对象或Arraybuffer对象）。

```
ws.onmessage = function(event){
  if(typeof event.data === String) {
    console.log("Received data string");
  }

  if(event.data instanceof ArrayBuffer){
    var buffer = event.data;
    console.log("Received arraybuffer");
  }
}
```

除了动态判断收到的数据类型，也可以使用 `binaryType` 属性，显式指定收到的二进制数据类型。

```
// 收到的是 blob 数据
ws.binaryType = "blob";
ws.onmessage = function(e) {
  console.log(e.data.size);
};

// 收到的是 ArrayBuffer 数据
ws.binaryType = "arraybuffer";
ws.onmessage = function(e) {
  console.log(e.data.byteLength);
};
```

### 常量

#### Ready state 常量

这些常量是 `readyState` 属性的取值，可以用来描述 WebSocket 连接的状态。

| 常量       | 值   | 描述                             |
| ---------- | ---- | -------------------------------- |
| CONNECTING | 0    | 连接还没开启。                   |
| OPEN       | 1    | 连接已开启并准备好进行通信。     |
| CLOSING    | 2    | 连接正在关闭的过程中。           |
| CLOSED     | 3    | 连接已经关闭，或者连接无法建立。 |

### 方法

### close()

关闭 WebSocket 连接或停止正在进行的连接请求。如果连接的状态已经是 `closed`，这个方法不会有任何效果

```
void close(in optional unsigned short code, in optional DOMString reason);
```

code 可选

一个数字值表示关闭连接的状态号，表示连接被关闭的原因。如果这个参数没有被指定，默认的取值是1000 （表示正常连接关闭）。 请看 CloseEvent 页面的 list of status codes来看默认的取值。

reason 可选

一个可读的字符串，表示连接被关闭的原因。这个字符串必须是不长于123字节的UTF-8 文本（不是字符）。

可能抛出的异常

- INVALID_ACCESS_ERR：选定了无效的code。
- SYNTAX_ERR：reason 字符串太长或者含有 `unpaired surrogates`。

### send()

通过 WebSocket 连接向服务器发送数据。

```
void send(in DOMString data);
void send(in ArrayBuffer data);
void send(in Blob data); 
```

data：要发送到服务器的数据。

可能抛出的异常：

- INVALID_STATE_ERR：当前连接的状态不是OPEN。
- SYNTAX_ERR：数据是一个包含 `unpaired surrogates` 的字符串。

发送文本的例子。

```
ws.send('your message');
```

发送 Blob 对象的例子。

```
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```

发送 ArrayBuffer 对象的例子。

```
// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```

## 服务端的实现

WebSocket 服务器的实现，可以查看维基百科的[列表](https://en.wikipedia.org/wiki/Comparison_of_WebSocket_implementations)。

常用的 Node 实现有以下三种。

- [Socket.IO](http://socket.io/)
- [µWebSockets](https://github.com/uWebSockets/uWebSockets)
- [WebSocket-Node](https://github.com/theturtle32/WebSocket-Node)

## 问答

### 和TCP、HTTP协议的关系

WebSocket 是基于 TCP 的独立的协议。它与 HTTP 唯一的关系是它的握手是由 HTTP 服务器解释为一个 Upgrade 请求。

WebSocket协议试图在现有的 HTTP 基础设施上下文中解决现有的双向HTTP技术目标；同样，它被设计工作在HTTP端口80和443，也支持HTTP代理和中间件，

HTTP服务器需要发送一个“Upgrade”请求，即101 Switching Protocol到HTTP服务器，然后由服务器进行协议转换。

### Sec-WebSocket-Key/Accept 的作用

前面提到了，`Sec-WebSocket-Key/Sec-WebSocket-Accept` 在主要作用在于提供基础的防护，减少恶意连接、意外连接。

作用大致归纳如下：

避免服务端收到非法的 websocket 连接（比如 http 客户端不小心请求连接 websocket 服务，此时服务端可以直接拒绝连接）

确保服务端理解 websocket 连接。因为 ws 握手阶段采用的是 http 协议，因此可能 ws 连接是被一个 http 服务器处理并返回的，此时客户端可以通过 `Sec-WebSocket-Key` 来确保服务端认识 ws 协议。（并非百分百保险，比如总是存在那么些无聊的 http 服务器，光处理 `Sec-WebSocket-Key`，但并没有实现 ws 协议。。。）

用浏览器里发起 ajax 请求，设置 header 时，`Sec-WebSocket-Key` 以及其他相关的 header 是被禁止的。这样可以避免客户端发送 ajax 请求时，意外请求`协议升级（websocket upgrade）`

可以防止反向代理（不理解 ws 协议）返回错误的数据。比如反向代理前后收到两次 ws 连接的升级请求，反向代理把第一次请求的返回给 cache 住，然后第二次请求到来时直接把 cache 住的请求给返回（无意义的返回）。

`Sec-WebSocket-Key` 主要目的并不是确保数据的安全性，因为 `Sec-WebSocket-Key`、`Sec-WebSocket-Accept` 的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。

### 数据掩码的作用

WebSocket 协议中，数据掩码的作用是增强协议的安全性。但数据掩码并不是为了保护数据本身，因为算法本身是公开的，运算也不复杂。除了加密通道本身，似乎没有太多有效的保护通信安全的办法。

那么为什么还要引入掩码计算呢，除了增加计算机器的运算量外似乎并没有太多的收益（这也是不少同学疑惑的点）。

答案还是两个字：安全。但并不是为了防止数据泄密，而是为了防止早期版本的协议中存在的`代理缓存污染攻击（proxy cache poisoning attacks）`等问题。

## 参考

- [WebSocket 教程——阮一峰](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
- [传统轮询、长轮询、服务器发送事件与WebSocket](http://blog.zhangruipeng.me/2015/10/22/Web-Connectivity/)
- [WebSocket API 文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)
- [RFC6455-- The WebSocket Protocol](https://tools.ietf.org/html/rfc6455)
- [WebSocket协议深入探究](https://mp.weixin.qq.com/s?__biz=MzIwNjQwMzUwMQ==&mid=2247485739&idx=1&sn=ef423ec4fb6b89cc5efd560117ea6931&chksm=97236be9a054e2ffe8d381f58675fced4ee35cf3ebdf2db24dbb05a22b091e99663b5fc1efe7&mpshare=1&scene=23&srcid=0123t2675RxJxXTuPKPLlkH7#rd)