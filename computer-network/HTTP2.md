## HTTP/2

### HTTP/1.x 的缺陷

HTTP/1.x 实现简单是以牺牲性能为代价的：

+   **客户端需要使用多个TCP连接才能实现并发和缩短延迟。**要想一个同时使用了HTML、CSS、JS的页面快速加载，那么不能只使用一个TCP而是使用多个TCP来发起并行请求。
+   **不会压缩请求和响应首部，从而导致不必要的网络流量。**在HTTP/1.x 时，经常会出现一个HTTP请求过大（首部字段过多、数据字段过大），甚至超出了TCP的大小限制，所以一个请求会被拆分成多个请求，而首部的一些字段会重复传输。除此以外，同个域名下的多个请求许多首部字段是相同的，这样会造成重复传输。

### 工作过程

1.  多路复用。多个线程的请求被分割开并放入一个连接中，进行传输。
2.  首部压缩。对重复的首部字段不再需要发送，降低传输规模。
3.  二进制传输。传输不基于文本而是二进制，在解析时更高效、传输时更加紧凑。
4.  优先级管理机制。对一些重要的请求分配高优先级。在旧版本中，一个复杂的页面加载需要多个TCP连接并发请求。但是可能最需要先加载的HTML反而最后返回，这时设置优先级就可以满足需要。

>   How does that work? High performance is achieved by using a special method – multiplexing of threads. Packets of several threads are mixed in one connection and then separated on arrival. Moreover, this protocol uses header compression as well as query prioritization, while the old protocols had few connections that passed on a single file at a time. HTTP/2 – the protocol constructed not on the text but in a binary format, and that’s why works faster. Also, the improved algorithm of distribution of priorities allows giving at first the most important files to browsers. In SPDY, the prioritization was done using a simpler algorithm.

### 特性

#### 1. 首部压缩

HTTP 1.1请求的大小有时甚至会大于TCP窗口的初始大小，因为它们需要等待带着ACK的响应回来以后才能继续被发送。HTTP/2对消息头采用**HPACK**（专为HTTP/2头部设计的压缩格式）进行压缩传输，能够节省消息头占用的网络的流量。而HTTP/1.x每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。

首部压缩策略：

+   HTTP/2在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键－值对，对于相同的数据，不再通过每次请求和响应发送；
+   首部表在HTTP/2的连接存续期内始终存在，由客户端和服务器共同渐进地更新；
+   每个新的首部键－值对要么被追加到当前表的末尾，要么替换表中之前的值。

如下，请求2只需要发送差异性的首部数据即可。

![首部压缩字段](https://pic3.zhimg.com/80/v2-1573194744d005dd110bbeac3a9b5246_1440w.jpg)

HPACK 压缩原理，可以看到有基于哈夫曼编码（Huffman）：

![HPACK](https://img2018.cnblogs.com/blog/1249/201812/1249-20181221093723730-1731019507.png)

#### 2. 服务端推送 Server Push

服务端可以在发送页面HTML时主动推送其它资源，而不用等到浏览器解析到相应位置，发起请求再响应。例如服务端可以主动把JS和CSS文件推送给客户端，而不需要客户端解析HTML时再发送这些请求。

服务端可以主动推送，客户端也有权利选择是否接收。如果服务端推送的资源已经被浏览器缓存过，浏览器可以通过发送RST_STREAM帧来拒收。主动推送也遵守同源策略，服务器不会随便推送第三方资源给客户端。

#### 3. 二进制分帧层

-   帧：HTTP/2 数据通信的最小单位消息。一般分为HEADERS 帧和 DATA 帧。
-   消息：指逻辑上的 HTTP 消息，例如请求和响应，消息由一个或多个帧组成

-   流：存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的标识符 ID 和可选的优先级信息。

 HTTP /1.x 的请求和响应报文，都是由起始行start-line、首部headers和实体body组成，各部分之间以文本换行符分隔（\n或\r\n）。HTTP/2 将请求和响应数据分割为更小的帧，并且采用二进制编码。

**HTTP/2 中，同域名下所有通信都在单个连接上完成，该连接可以承载任意数量的双向数据流。**每个数据流都以消息的形式发送，而消息又由多个帧组成。多个帧之间可以乱序发送，根据帧首部的流ID可以重新组装。

![](http://upload-images.jianshu.io/upload_images/5578817-69e837e5dac1c652.png)

#### 4. 多路复用 MultiPlexing

在 HTTP/2 中，有了二进制分帧之后，HTTP/2 不再依赖 TCP 连接去实现多请求并行了，在 HTTP/2中：

+   同域名下所有通信都在单个TCP连接上完成。
+   单个连接可以承载任意数量的双向数据流。
+   数据流以消息的形式发送，而消息又由一个或多个帧组成，多个帧之间可以乱序发送，因为根据帧首部的流标识可以重新组装。

这一特性，使性能有了极大提升：

+   **同个域名只需要占用一个 TCP 连接**，消除了因多个 TCP 连接而带来的延时和内存消耗。
+   单个连接上可以并行交错的请求和响应，之间互不干扰。
+   在HTTP/2中，每个请求都可以带一个31bit的优先值，0表示最高优先级， 数值越大优先级越低。有了这个优先值，客户端和服务器就可以在处理不同的流时采取不同的策略，以最优的方式发送流、消息和帧。

#### 5. 二进制传输

HTTP/2 采用**二进制格式**传输帧，而非 HTTP 1.x 的文本格式，二进制协议解析起来更高效。

### 优点

1.  由于优秀的优先级管理机制，可以快速处理使用HTML、CSS、 JavaScript 和大图片文件等作展示的复杂页面；
2.  强制使用TLS来保证传输安全；
3.  解析数据低开销。由于传输是二进制的，解析更高效；
4.  降低失败率，如图片加载失败；
5.  减少网络的压力。二进制传输，更紧凑；首部压缩，减少总请求规模；
6.  有效利用网络资源；
7.  多重传输；
8.  在客户端和服务器之间进行更高效的数据处理。。

>   +   possibility to speed up the work of complex pages that use HTML, CSS, JavaScript, a large number of pictures due to the competent prioritization;
>   +   obligatory use of TLS allows providing maximum protection;
>   +   low cost of parsing data;
>   +   fewer mistakes;
>   +   less strain on the network;
>   +   efficient use of network resources;
>   +   multiple transmission;
>   +   more efficient data processing between client and server.



### VS HTTP/1.1

HTTP/2兼容旧版本的语义、模式等，主要的改变在于多端之间数据包的创建和传输方式。

1.  多个请求复用同一个TCP连接，降低延迟；
2.  请求可以设置优先级；
3.  综合上，首部字段规模缩小；
4.  服务端可以主动向客户端推送资源，比如提前发送一个下一个页面，这样客户端就不用再次发起请求。

通过以上几个改变，缩短了网络延迟，提高了页面加载速度。

>   The server can transfer data to the client even if the browser has not yet requested it, but the browser must display the page completely. Additional queries can be multiplexed (combined queries or responses) and forwarded in a conveyor manner (multiple queries without waiting for the corresponding responses) with a single TCP connection. These improvements reduce latency and lead to better page loading speed.
>
>   Also, the HTTP/2 protocol has the following features:
>
>   +   multiple requests can be sent over a single TCP connection, and responses can be received in any order;
>   +   the client can set priorities for the server – which type of resources are more vital for it than the others;
>   +   the size of the HTTP header can be reduced;
>   +   the server can send data to the client that the client has not yet requested, for example, based on what page the users are opening next.



[HTTP/1.1与HTTP/2的使用对比](https://http2.akamai.com/demo)

### VS HTTTPS

HTTP/2规定上是可以不使用TLS的，但现在大多浏览器都规定只支持带有TLS的HTTP/2



### Q&A

**Q1：为什么基于二进制传输比基于文本要快、成本低？**

A：如一个数字77，用二进制存储传输需要7 bit；而若是字符串，按照UTF-8码，则需2 bytes=16 bit，如是UTF-16，则是4 bytes=32 bit

……



**Q2：支持TLS的HTTP/2浪费了多路复用机制吗？**

A：

### 参考

1.  https://www.h2check.org
2.  https://zhuanlan.zhihu.com/p/26559480
3.  https://www.jianshu.com/p/e57ca4fec26f