###  计网基础

### DNS解析步骤 

1️⃣查看浏览器缓存

2️⃣ 查看操作系统缓存

3️⃣查看host文件

4️⃣ 询问本地DNS服务器（服务器本地缓存 —> 根域名服务器(.) —> 顶级域名服务器(.com) —> 权威DNS服务器(server.com)）

### MSS、MTU相关

- `MTU`：一个网络包的最大长度，以太网中一般为 `1500` 字节。
- `MSS`：除去 IP 和 TCP 头部之后，一个网络包所能容纳的 TCP 数据的最大长度。

![数据包分割](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/12.jpg)

#### 为什么有了MTU还需要有MSS

**MTU 作用于链路层**，决定单帧最大大小，包括 TCP/IP 头和数据。

**MSS 作用于 TCP 层**，决定单个 TCP 段的最大数据负载，以避免 IP 分片，提高传输效率。

- 作用：1️⃣防止IP分片，提高传输效率， 2️⃣适应不同的网络环境

### 内核接受网络包流程

#### NAPI（New API）

通过**减少中断频率**，结合**中断驱动和轮询机制**，提高网络数据包的处理效率并减少系统资源的开销。

- 传统网络终端弊端：<u>”中断风暴“</u>
- NAPI优化：在接收到数据包时，最初仍通过中断方式通知内核。进入内核后，<u>中断被禁用</u>，改用<u>轮询方式批量处理数据包</u>（软中断）。

#### 网络协议栈解析处理

- **传输层—>socket层**：根据四元组`{源 IP、源端口、目的 IP、目的端口}` 作为标识，找出对应的 Socket，并把数据放到 Socket 的接收缓冲区

- 应用层程序调用 Socket 接口，将内核的 <u>Socket 接收缓冲区的数据「拷贝」到应用层的缓冲区</u>，然后唤醒用户进程。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E6%94%B6%E5%8F%91%E6%B5%81%E7%A8%8B.png)

#### [Linux发送网络包流程](https://xiaolincoding.com/network/1_base/how_os_deal_network_package.html#linux-%E5%8F%91%E9%80%81%E7%BD%91%E7%BB%9C%E5%8C%85%E7%9A%84%E6%B5%81%E7%A8%8B)(细看)

如上图右侧所示

> 首先，应用程序会调用 Socket 发送数据包的接口，由于这个是系统调用，所以会从用户态陷入到内核态中的 Socket 层，内核会申请一个内核态的 sk_buff 内存，**将用户待发送的数据拷贝到 sk_buff 内存，并将其加入到发送缓冲区**。
>
> 接下来，网络协议栈从 Socket 发送缓冲区中取出 sk_buff，并按照 TCP/IP 协议栈从上到下逐层处理。
>
> 如果使用的是 TCP 传输协议发送数据，那么**先拷贝一个新的 sk_buff 副本** ，这是因为 sk_buff 后续在调用网络层，最后到达网卡发送完成的时候，这个 sk_buff 会被释放掉。而 TCP 协议是支持丢失**重传**的，在收到对方的 ACK 之前，这个 sk_buff 不能被删除。所以内核的做法就是每次调用网卡发送的时候，实际上传递出去的是 sk_buff 的一个拷贝，等收到 ACK 再真正删除





## 打开网页的过程

- 解析URL：分析 URL 所需要使用的传输协议和请求的资源路径。如果输入的 URL 中的协议或者主机名不合法，将会把地址栏中输入的内容传递给搜索引擎。如果没有问题，浏览器会检查 URL 中是否出现了非法字符，则对非法字符进行转义后在进行下一过程。
- 缓存判断：浏览器会判断所请求的资源是否在缓存里，如果请求的资源在缓存里且没有失效，那么就直接使用，否则向服务器发起新的请求。
- DNS解析：如果资源不在本地缓存，首先需要进行DNS解析。浏览器会向本地DNS服务器发送域名解析请求，本地DNS服务器会逐级查询，最终找到对应的IP地址。
- 获取MAC地址：当浏览器得到 IP 地址后，数据传输还需要知道目的主机 MAC 地址，因为应用层下发数据给传输层，TCP 协议会指定源端口号和目的端口号，然后下发给网络层。网络层会将本机地址作为源地址，获取的 IP 地址作为目的地址。然后将下发给数据链路层，数据链路层的发送需要加入通信双方的 MAC 地址，本机的 MAC 地址作为源 MAC 地址，目的 MAC 地址需要分情况处理。通过将 IP 地址与本机的子网掩码相结合，可以判断是否与请求主机在同一个子网里，如果在同一个子网里，可以使用 APR 协议获取到目的主机的 MAC 地址，如果不在一个子网里，那么请求应该转发给网关，由它代为转发，此时同样可以通过 ARP 协议来获取网关的 MAC 地址，此时目的主机的 MAC 地址应该为网关的地址。
- 建立TCP连接：主机将使用目标 IP地址和目标MAC地址发送一个TCP SYN包，请求建立一个TCP连接，然后交给路由器转发，等路由器转到目标服务器后，服务器回复一个SYN-ACK包，确认连接请求。然后，主机发送一个ACK包，确认已收到服务器的确认，然后 TCP 连接建立完成。
- HTTPS 的 TLS 四次握手：如果使用的是 HTTPS 协议，在通信前还存在 TLS 的四次握手。
- 发送HTTP请求：连接建立后，浏览器会向服务器发送HTTP请求。请求中包含了用户需要获取的资源的信息，例如网页的URL、请求方法（GET、POST等）等。
- 服务器处理请求并返回响应：服务器收到请求后，会根据请求的内容进行相应的处理。例如，如果是请求网页，服务器会读取相应的网页文件，并生成HTTP响应。





# HTTP

## HTTP基础

### 常见状态码

`1xx` 类状态码属于**提示信息**，是协议处理中的一种中间状态，实际用到的比较少。

`2xx` 类状态码表示服务器**成功**处理了客户端的请求，也是我们最愿意看到的状态。

- 「**200 OK**」是最常见的成功状态码，表示一切正常。如果是非 `HEAD` 请求，服务器返回的响应头都会有 body 数据。
- 「**204 No Content**」也是常见的成功状态码，与 200 OK 基本相同，但响应头没有 body 数据。
- 「**206 Partial Content**」是应用于 HTTP 分块下载或断点续传，表示响应返回的 body 数据并不是资源的全部，而是其中的一部分，也是服务器处理成功的状态。

`3xx` 类状态码表示客户端请求的资源发生了变动，需要客户端用新的 URL 重新发送请求获取资源，也就是**重定向**。

- 「**301 Moved Permanently**」表示永久重定向，说明请求的资源已经不存在了，需改用新的 URL 再次访问。
- 「**302 Found**」表示临时重定向，说明请求的资源还在，但暂时需要用另一个 URL 来访问。

301 和 302 都会在响应头里使用字段 `Location`，指明后续要跳转的 URL，浏览器会自动重定向新的 URL。

- 「**304 Not Modified**」不具有跳转的含义，表示资源未修改，重定向已存在的缓冲文件，也称缓存重定向，也就是告诉客户端可以继续使用缓存资源，用于缓存控制。

`4xx` 类状态码表示客户端发送的**报文有误**，服务器无法处理，也就是错误码的含义。

- 「**400 Bad Request**」表示客户端请求的报文有错误，但只是个笼统的错误。
- **401 Unauthorized**
- 「**403 Forbidden**」表示服务器禁止访问资源，并不是客户端的请求出错。
- 「**404 Not Found**」表示请求的资源在服务器上不存在或未找到，所以无法提供给客户端。

`5xx` 类状态码表示客户端请求报文正确，但是**服务器处理时内部发生了错误**，属于服务器端的错误码。

- 「**501 Not Implemented**」表示客户端请求的功能还不支持，类似“即将开业，敬请期待”的意思。
- 「**502 Bad Gateway**」通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问后端服务器发生了错误。
- 「**503 Service Unavailable**」表示服务器当前很忙，暂时无法响应客户端，类似“网络服务正忙，请稍后重试”的意思。
- **504 Gateway Time-out**：作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器收到响应。



### 常见字段

 	 	





## HTTP缓存

![image-20250113下午115124506](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250113%E4%B8%8B%E5%8D%88115124506.png)



## HTTPS

### 数字证书颁发和验证

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250114%E4%B8%8A%E5%8D%88120830275.png" alt="image-20250114上午120830275" style="zoom:50%;" />

- 首先 CA 会把持有者的公钥、用途、颁发者、有效时间等信息打成一个包，然后对这些信息进行 Hash 计算，得到一个 Hash 值；
- 然后 CA 会使用自己的私钥将该 Hash 值加密，生成 Certificate Signature，也就是 CA 对证书做了签名；
- 最后将 Certificate Signature 添加在文件证书上，形成数字证书；



### TLS/RSA握手

传统的 TLS 握手基本都是使用 RSA 算法来实现密钥交换

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/https/https_rsa.png)

- **RAS算法缺陷**：

  **使用 RSA 密钥协商算法的最大问题是不支持前向保密**。

  因为客户端传递随机数pre_master（用于生成对称加密密钥的条件之一）给服务端时使用的是公钥加密的，服务端收到后，会用私钥解密得到随机数。所以一旦服务端的私钥泄漏了，过去被第三方截获的所有 TLS 通讯密文都会被破解。

  为了解决这个问题，后面就出现了 ECDHE 密钥协商算法





### ECDHE握手

DH ——〉DHE ——〉ECDHE

#### DH算法

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/https/dh%E7%AE%97%E6%B3%95.png" alt="img" style="zoom:50%;" />

A、B为公钥，  a、b为私钥，g、p公有，K为对称加密密钥

攻击者只知道 g 、 p 、 $g^a \mod p$ 和 $g^b \mod p$ 。

攻击者需要计算 $g^{ab} \mod p$ ，这等价于求解 **离散对数问题**：

- 给定 $g^a \mod p$ ，推导 a 是困难的（离散对数问题的难度）。

- 如果素数 p 足够大，离散对数问题在现有算法下是不可行的，确保了 DH 算法的安全性。

#### DHE算法

static DH 算法里有一方的私钥是静态的，也就说每次密钥协商的时候有一方的私钥都是一样的，一般是服务器方固定，即 a 不变，客户端的私钥则是随机生成的。

DHE算法让双方的<u>私钥在每次密钥交换通信时，都是随机生成的</u>、临时的，这个方式也就是 DHE 算法，E 全称是 ephemeral（临时性的）。

**保证了向前安全**



- 缺点：性能差

#### ECDHE算法

> [!NOTE]
>
> 1. C-〉S：客户端随机数，支持的密码套件列表、TLS版本
> 2. S-〉C：确认TLS版本、服务端随机数、选择的密码套件（椭圆曲线）、服务端证书、椭圆曲线共钥
> 3. C-〉S：客户端椭圆曲线共钥、**Encrypted Handshake Message**（之前发送的数据做一个摘要，再用对称密钥加密一下）
> 4. S-〉C：**Encrypted Handshake Message**

- 双方事先确定好使用哪种椭圆曲线，和曲线上的基点 `G`，这两个参数都是公开的；
- 双方各自随机生成一个随机数作为**私钥`d`**，并与基点 G相乘得到**公钥Q**（`Q = dG`），此时小红的公私钥为 Q1 和 d1，小明的公私钥为 Q2 和 d2；
- 双方交换各自的公钥，最后小红计算点（x1，y1） = d1Q2，小明计算点（x2，y2） = d2Q1，由于椭圆曲线上是可以满足乘法交换和结合律，所以 d1Q2 = d1d2G = d2d1G = d2Q1 ，因此**双方的 x 坐标是一样的，所以它是共享密钥，也就是会话密钥**。



### HTTPS优化

#### 协议优化

**RSA  ——〉ECDHE**：

- 保证向前安全

> 在 ECDHE 协议中，客户端和服务器在每次会话开始时都会生成一个临时的椭圆曲线密钥对：
>
> ​	• 私钥：随机生成的临时值 $d_A$ 和 $d_B$ 。
>
> ​	• 公钥：通过椭圆曲线点倍乘 $Q_A = d_A \cdot G$ 和 $Q_B = d_B \cdot G$ 计算得到。

- 客户端可以在 TLS 协议的第 3 次握手后，第 4 次握手前，发送加密的应用数据

**TLS1.2 ——〉TLS1.3**

- 握手时延从`2RTT`降到`1RTT`（客户端可以在 TLS 协议的第 3 次握手后，第 4 次握手前，发送加密的应用数据）



#### 证书优化

- **证书传输优化**：对于服务器的证书应该选择椭圆曲线（ECDSA）证书，而不是 RSA 证书，因为在相同安全强度下， ECC 密钥长度比 RSA 短的多。
- **证书验证优化**：
  - `CRL` 称为证书吊销列表（*Certificate Revocation List*），定时更新，实时性较差
  - `OCSP`在线证书状态协议（*Online Certificate Status Protocol*）来查询证书的有效性，它的工作方式是**向 CA 发送查询请求，让 CA 返回证书的有效状态**。：受网络因素影响较大
  - `OCSP Stapling`:为了解决这一个网络开销，就出现了 OCSP Stapling，其原理是：<u>服务器</u>向 CA 周期性地查询证书状态，获得一个带有**时间戳和签名**的响应结果并缓存它。<u>当有客户端发起连接请求时，服务器会把这个「响应结果」在 TLS 握手过程中发给客户端</u>。由于有签名的存在，服务器无法篡改，因此客户端就能得知证书是否已被吊销了，这样客户端就不需要再去查询。



#### 会话复用

- Session ID

完成握手之后，双方保存唯一的会话ID，之后客户端将会话ID发送给服务端验证

<u>缺点：耗费服务端存储，分布式/负载均衡下可能无法命中</u>

- Session Ticket

在初次握手完成后，服务器生成一个加密的会话票据，<u>其中包含会话密钥等信息（加密算法、共享密钥、会话参数、到期时间）</u>。票据由服务器使用其<u>密钥进行加密</u>，然后发送给客户端保存。

- 两者都**不具备向前安全性**



#### Pre-shared Key

在使用TLS1.3与Session Ticket下，支持0-RTT重连

- 可能遭受**重放攻击**，服务器需要设计防重放机制（如基于时间戳或序列号）。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250116%E4%B8%8A%E5%8D%8810235141.png" alt="image-20250116上午10235141" style="zoom:50%;" />

> [!NOTE]
>
> - 为什么是TLS1.3 ， TLS1.2不行？
>
> 
>
> - 为什么是Session Ticket ，Session Id不行？
>
> 







## HTTP2

### 头部压缩

**HPACK算法**：静态字典 + 动态字典 + Huffman编码

### 二进制帧

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250116%E4%B8%8B%E5%8D%88105429998.png" alt="image-20250116下午105429998" style="zoom:50%;" />

- 帧类型：

  <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250116%E4%B8%8B%E5%8D%88105604510.png" alt="image-20250116下午105604510" style="zoom:50%;" />

- 帧标志：

  可以保存 8 个标志位，用于携带简单的控制信息，比如：

  - **END_HEADERS** 表示头数据结束标志，相当于 HTTP/1 里头后的空行（“\r\n”）；
  - **END_Stream** 表示单方向数据发送结束，后续不会再有数据帧。
  - **PRIORITY** 表示流的优先级；

### stream并发传输

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250116%E4%B8%8B%E5%8D%88105740039.png" alt="image-20250116下午105740039" style="zoom:50%;" />

一个Message对应一个请求或响应，<u>不同stream下的frame可以乱序，同一个stream下的frame不可以乱序</u>

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250116%E4%B8%8B%E5%8D%88105935130.png" alt="image-20250116下午105935130" style="zoom:40%;" />

- 并发比HTTP1.1好很多：**因为当 HTTP/2 实现 100 个并发 Stream 时，只需要建立一次 TCP 连接，而 HTTP/1.1 需要建立 100 个 TCP 连接，每个 TCP 连接都要经过 TCP 握手、慢启动以及 TLS 握手过程，这些都是很耗时的。**

- 当 Stream ID 耗尽时，需要发一个控制帧 `GOAWAY`，用来关闭 TCP 连接

- 在 Nginx 中，可以通过 `http2_max_concurrent_Streams` 配置来设置 Stream 的上限，默认是 128 个

- HTTP/2 还可以对每个 Stream 设置不同**优先级**，帧头中的「标志位」可以设置优先级，比如客户端访问 HTML/CSS 和图片资源时，希望服务器先传递 HTML/CSS，再传图片，那么就可以通过设置 Stream 的优先级来实现，以此提高用户体验。

  

### 服务器推送

服务器主动的推送，使用的是偶数号 Stream。服务器在推送资源时，会通过 `PUSH_PROMISE` 帧传输 HTTP 头部，并通过帧中的 `Promised Stream ID` 字段告知客户端，接下来会在哪个偶数号 Stream 中发送包体。

## HTTP3

> - HTTP/1.1 中的管道（ pipeline）虽然解决了请求的队头阻塞，但是**没有解决响应的队头阻塞**，因为服务端需要按顺序响应收到的请求，如果服务端处理某个请求消耗的时间比较长，那么只能等响应完这个请求后， 才能处理下一个请求，这属于 HTTP 层队头阻塞。
> - HTTP/2 虽然通过多个请求复用一个 TCP 连接解决了 HTTP 的队头阻塞 ，但是**一旦发生丢包，就会阻塞住所有的 HTTP 请求**，这属于 TCP 层队头阻塞。

QUIC 有以下 3 个特点。

- 无队头阻塞
- 更快的连接建立
- 连接迁移
- 

## RPC与HTTP区别



![image-20250129下午112629277](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250129%E4%B8%8B%E5%8D%88112629277.png)

- **RPC** 更注重 **性能** 和 **内部服务的高效通信**，适合微服务架构。
- **HTTP** 更注重 **通用性** 和 **跨平台兼容性**，适合开放 API 和前后端交互。

## RESTful API 与 HTTP 风格的区别

RESTful API 和 HTTP 风格虽然密切相关，但它们有一些本质上的区别。以下是二者的核心差异：

---

### 1. RESTful API 的核心理念

RESTful API 是一种基于 **REST（Representational State Transfer）架构** 的设计风格，其目的是使 API 的设计更加清晰、简洁和易于使用。它的设计主要关注以下原则：

- **资源**：通过唯一的 URL 表示资源，每个资源都有唯一的路径。
- **状态无关性**：每次请求都应包含完成操作所需的所有信息，服务器不维护客户端状态。
- **统一接口**：通过标准的 HTTP 方法（如 GET、POST、PUT、DELETE）来操作资源。
- **表现形式**：资源的表现形式可以是 JSON、XML 等格式。
- **客户端-服务器解耦**：客户端与服务器的实现逻辑完全独立，客户端只需通过 API 操作资源。

---

### 2. HTTP 风格

HTTP 是一种协议，提供了数据在客户端和服务器之间传输的规则。HTTP 风格指的是基于 HTTP 协议的请求和响应机制，但不一定遵循 REST 的架构原则。

HTTP 风格通常没有明确的资源概念，更关注的是 **如何调用服务的接口**，而非如何设计资源。常见的 HTTP 风格特点包括：

- 请求路径并不总是以资源为中心。
- URL 中可能嵌入操作动词（例如 `/getUser`、`/createShop`）。
- 方法选择不严格，通常都用 `POST`，甚至用 `GET` 执行修改操作。
- 状态可能依赖于服务器端的会话管理。
- 数据格式可能没有一致性，甚至直接返回 HTML，而非 JSON 或 XML。

---

### 3. 具体比较

| **特点**      | **RESTful API**                                              | **HTTP 风格**                               |
| ------------- | ------------------------------------------------------------ | ------------------------------------------- |
| **设计理念**  | 基于资源，每个 URL 对应一个资源。                            | 基于操作，每个 URL 通常对应一个操作。       |
| **URL 命名**  | 资源路径（如 `/users`、`/orders`）。                         | 操作路径（如 `/getUser`、`/createOrder`）。 |
| **HTTP 方法** | 严格遵守，如 GET（查）、POST（增）、PUT（改）、DELETE（删）。 | 不严格，通常所有操作都用 POST 或 GET。      |
| **数据格式**  | 通常是 JSON 或 XML，表现形式独立于实现。                     | 数据格式可能不一致，甚至直接返回 HTML。     |
| **状态管理**  | 无状态，客户端每次请求都携带完整信息。                       | 可能依赖于服务器端会话管理。                |
| **扩展性**    | 高扩展性，可轻松支持新资源或新操作。                         | 难以扩展，需要为每个新操作添加特定路径。    |

---

### 4. 示例对比

#### RESTful API 示例

- **获取用户信息**

  ```http
  GET /users/123
  ```

- **创建新用户**

```http
POST /users
{
 "name": "Alice",
 "email": "alice@example.com"

}
```



#### HTTP 风格示例

- **获取用户信息**

```http
POST /getUser
{
 "id": 123
}
```

通过 POST 方法请求用户信息。

- **创建新用户**

```http
POST /createUser
{
 "name": "Alice",
 "email": "alice@example.com"
}
```

### 总结

- **关键区别总结**
  - **RESTful API** 强调**资源为中心**，使用统一接口和标准的 HTTP 方法。
  - **HTTP 风格**更像是对服务接口的简单调用，强调**操作为中心**，但不严格遵循 REST 的架构原则。

- **6. 关系总结**
  - **RESTful API** 是一种架构风格**，用于规范化设计和交互方式，使得接口更加清晰、统一。**
  - **HTTP 是通信协议**，RESTful API 通常基于 HTTP，但 HTTP 风格不一定遵循 REST 的设计原则。

**简单来说**：RESTful API 利用 HTTP 协议并通过设计原则提升了接口的规范性和易用性，而 HTTP 风格只是简单利用了协议本身，没有统一的设计理念。







# TCP

## 基础认识

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250127%E4%B8%8A%E5%8D%88121800845.png" alt="image-20250127上午121800845" style="zoom:50%;" />



### 序列号的作用

去重、按序接收、标识哪些数据包是被对方接受了的

### 为什么不是两次/四次握手

- 避免历史链接
- 同步序列号
- 避免服务端资源浪费

### 初始化序列号为什么要不一样

- 避免历史数据（相同四元组）
- 防止第三方伪造数据

### IP层会以MTU分片为什么TCP层还要MSS

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250117%E4%B8%8B%E5%8D%8894057892.png" alt="image-20250117下午94057892" style="zoom:50%;" />

- **那么当如果一个 IP 分片丢失，整个 IP 报文的所有分片都得重传**

当某一个 IP 分片丢失后，接收方的 IP 层就无法组装成一个完整的 TCP 报文（头部 + 数据），也就无法将数据报文送到 TCP 层，所以接收方不会响应 ACK 给发送方，因为发送方迟迟收不到 ACK 确认报文，所以会触发超时重传，就会重发「整个 TCP 报文（头部 + 数据）」。



### 什么是SYN攻击，如何避免

- 增大SYN队列长度
- 开启syncookies
- 减少SYN + ACK重传次数



### close与shutdown区别

![image-20250121下午113152974](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250121%E4%B8%8B%E5%8D%88113152974.png)

- `shutdown`常用于实现半关闭状态

```cpp
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
connect(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr));

// 关闭写方向，通知对方数据发送完毕，但仍然可以接收数据
shutdown(sockfd, SHUT_WR);

// 处理接收的数据
char buffer[1024];
recv(sockfd, buffer, sizeof(buffer), 0);

// 最终完全关闭套接字
close(sockfd);
```

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250117%E4%B8%8B%E5%8D%88104032338.png" alt="image-20250117下午104032338" style="zoom:25%;" />

- `close`:每次一个进程或线程调用 close，引用计数会减少。**只有当引用计数归零时，内核才会释放套接字及其资源**。



<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250117%E4%B8%8B%E5%8D%88104020065.png" alt="image-20250117下午104020065" style="zoom:25%;" />



### TIME_WAIT相关

#### TIME_WAIT有什么用，为什么要是2MSL



#### TIME_WAIT过多的危害

- 系统资源
- 端口资源
  - 客户端：TIME_WAIT过多导致端口用完了，如果 目的ip + 目的port一样则可以复用、反之则不能
  - 服务端：TIME_WAIT过多并不会导致端口资源受限

#### 如何优化TIME_WAIT

- 打开 net.ipv4.tcp_tw_reuse 和 net.ipv4.tcp_timestamps 选项；
- net.ipv4.tcp_max_tw_buckets
- 程序中使用 SO_LINGER ，应用强制使用 RST 关闭。

#### 服务端出现大量TIME_WAIT的原因

- **没有使用HTTP长链接**：任意一方没有开启 HTTP Keep-Alive，都会导致**服务端**在处理完一个 HTTP 请求后，就主动关闭连接
- **长链接超时**：`nginx`的`keepalive_timeout`参数
- **长链接数量达到上限**：`nginx` 的 `keepalive_requests` 参数



#### 服务端出现大量CLOSE_WAIT

https://mp.weixin.qq.com/s?__biz=MzU3Njk0MTc3Ng==&mid=2247486020&idx=1&sn=f7cf41aec28e2e10a46228a64b1c0a5c&scene=21#wechat_redirect



### TCP保活机制

- 开启：socket接口需设置`SO_KEEPALIVE`

- 一段时间没有活跃，开始保活机制，发送探测报文

  ```
  tcp_keepalive_time=7200：表示保活时间是 7200 秒（2小时），也就 2 小时内如果没有任何连接相关的活动，则会启动保活机制
  tcp_keepalive_intvl=75：表示每次检测间隔 75 秒；
  tcp_keepalive_probes=9：表示检测 9 次无响应，认为对方是不可达的，从而中断本次的连接
  ```

1. **对方主机仍然在线，但应用已关闭或端口未打开**：内核会直接回复一个 **RST 报文**，表明当前端口无法处理该连接。

2. **对方主机宕机**：没有响应



### socket下的tcp

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8A%E5%8D%88122720640.png" alt="image-20250122上午122720640" style="zoom:50%;" />

服务端接收到了 FIN 报文，TCP 协议栈会为 FIN 包插入一个文件结束符 `EOF` 到接收缓冲区中，应用程序可以通过 `read` 调用来感知这个 FIN 包。



## TCP的可靠性

### 超时重传与快速重传

**超时重传：**

- 超时重传时间以 `RTO` （Retransmission Timeout 超时重传时间）表示，**超时重传时间 RTO 的值应该略大于报文往返 RTT 的值**
- RTO时间应该是动态变化的，**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送**

**快速重传：**

- 一般方法：三个相同ACK重传，但是不知道改重传一个还是后面的所有

- `SACK`(Selective Acknowledgment)：

  这种方式需要在 TCP 头部「选项」字段里加一个 `SACK` 的东西，它**可以将已收到的数据的信息发送给「发送方」**

  <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8B%E5%8D%8815506456.png" alt="image-20250122下午15506456" style="zoom:50%;" />

- Duplicate SACK(`D-SACK`)

  **使用了 SACK 来告诉「发送方」有哪些数据被重复接收了**，<u>是因为ACK丢失</u>，<u>还是发送发网络延时</u>

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8B%E5%8D%8820513460.png" alt="image-20250122下午20513460" style="zoom:50%;" />

### 滑动窗口

#### 窗口关闭的风险

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8B%E5%8D%8821937922.png" alt="image-20250122下午21937922" style="zoom:50%;" />

#### 窗口探测报文

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8B%E5%8D%8822300506.png" alt="image-20250122下午22300506" style="zoom:50%;" />

窗口探测的次数一般为 3 次，每次大约 30-60 秒（不同的实现可能会不一样）。如果 3 次过后接收窗口还是 0 的话，有的 TCP 实现就会发 `RST` 报文来中断连接。

#### 糊涂窗口综合症

- 什么是：接收方的接收窗口（rwnd）很小，无法容纳一个足够大的数据段。一旦有少量空间释放（例如几个字节），接收方立即通知发送方更新窗口大小。

- 怎么解决：

  - Nagle算法：

    ```c++
    if 有数据要发送 {
        if 可用窗口大小 >= MSS and 可发送的数据 >= MSS {
        	立刻发送MSS大小的数据
        } else {
            if 有未确认的数据 {
                将数据放入缓存等待接收ACK
            } else {
                立刻发送数据
            }
        }
    }
    ```

    





### 拥塞控制

#### 有了流量控制为什么还要有拥塞控制？

**TCP 流量控制**是一种**基于接收方能力**的机制，用于**防止发送方发送过快，导致接收方处理不过来**，从而避免数据丢失。TCP 通过 **“滑动窗口协议”** 来实现流量控制。

**在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重**



#### 拥塞控制算法

- 慢启动

- 拥塞避免：每当收到一个 ACK时，cwnd增加1/cwnd

- 拥塞发生

  - 发生超时重传：1️⃣`ssthresh` 设为 `cwnd/2` 2️⃣`cwnd` 重置为 `1` （是恢复为 cwnd 初始化值，假定 cwnd 初始化值 1）
  - 发生快速重传：1️⃣`cwnd = cwnd/2`，2️⃣`ssthresh = cwnd`，3️⃣进入快速恢复

- 快速恢复

  快速重传和快速恢复算法一般同时使用，快速恢复算法是认为，你还能收到 3 个重复 ACK 说明网络也不那么糟糕，所以没有必要像 `RTO` 超时那么强烈。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250123%E4%B8%8B%E5%8D%8883046326.png" alt="image-20250123下午83046326" style="zoom:50%;" />





## 端口复用相关

### TCP与UDP能否绑定相同端口



### 多个TCP服务绑定能否同一个端口？



SO_REUSEADDR 作用：**如果当前启动进程绑定的 IP+PORT 与处于TIME_WAIT 状态的连接占用的 IP+PORT 存在冲突，但是新启动的进程使用了 SO_REUSEADDR 选项，那么该进程就可以绑定成功**

### 客户端端口重复使用？

#### 多个客户端可以 bind 同一个端口吗？

而如果我们想自己指定连接的端口，就可以用 bind 函数来实现：<u>客户端先通过 bind 函数绑定一个端口，然后调用 connect 函数就会跳过端口选择的过程了，转而使用 bind 时确定的端口</u>。



#### 客户端TIME_WAIT过多

举个例子，假设客户端已经与服务器建立了一个 TCP 连接，并且这个状态处于 TIME_WAIT 状态：

```bash
客户端地址:端口           服务端地址:端口         TCP 连接状态
192.168.1.100:2222      172.19.11.21:8888     TIME_WAIT
```

然后客户端又与该服务器（172.19.11.21:8888）发起了连接，**在调用 connect 函数时，内核刚好选择了 2222 端口，接着发现已经被相同四元组的连接占用了：**

- 如果**没有开启** `net.ipv4.tcp_tw_reuse` 内核参数，那么内核就会选择下一个端口，然后继续判断，直到找到一个没有被相同四元组的连接使用的端口， 如果端口资源耗尽还是没找到，那么 connect 函数就会返回错误。
- 如果**开启**了 `net.ipv4.tcp_tw_reuse` 内核参数，就会判断该四元组的连接状态是否处于 TIME_WAIT 状态，**如果连接处于 TIME_WAIT 状态并且该状态持续的时间超过了 1 秒，那么就会重用该连接**，于是就可以使用 2222 端口了，这时 connect 就会返回成功。

再次提醒一次，开启了 `net.ipv4.tcp_tw_reuse` 内核参数，是客户端（连接发起方） 在调用 connect() 函数时才起作用，所以在服务端开启这个参数是没有效果的。







## [tcp思维导图](../笔试面试总结/TCP建立链接前后异常问题.xmind)

# IP



- DHCP客户端端口：68， DHCP服务端端口：67
- DNS端口：53



## ICMP

- ICMP功能：**确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。**

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8830530839.png" alt="image-20250124下午30530839" style="zoom:50%;" />

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250123%E4%B8%8B%E5%8D%88111812983.png" alt="image-20250123下午111812983" style="zoom:50%;" />

## ping的工作原理

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8821915857.png" alt="image-20250124下午21915857" style="zoom:50%;" />

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8822115556.png" alt="image-20250124下午22115556" style="zoom:50%;" />



### 为什么断网了还能ping通127.0.0.1

当发现IP是回环地址时会讲数据发送到**`input_pkt_queue` 的 链表 中。这个链表，其实是所有网卡共享的，上面挂着发给本机的各种消息。消息被发送到这个链表后，会再触发一个软中断**







# 其他

## Cookie、session、JWT

### JWT构成

1. Header

头部包含 Token 的类型（JWT）和签名算法（如 HS256、RS256）。

2. Payload

载荷存储 **声明（Claims）**，可以包含：

​	•	**标准声明**（Registered Claims）：

​		•	iss（Issuer）：签发者

​		•	sub（Subject）：主题

​		•	exp（Expiration Time）：过期时间

​		•	iat（Issued At）：签发时间

​	•	**私有声明**（Custom Claims）：可以存储用户 ID、权限等自定义数据。

3. Signature

计算方式：

```json
HMACSHA256(
    base64UrlEncode(Header) + "." + base64UrlEncode(Payload),
    secret
)
```



<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250129%E4%B8%8B%E5%8D%88105247070.png" alt="image-20250129下午105247070" style="zoom:50%;" />

### 认证流程

1. **用户登录**：

用户提供用户名和密码，服务器验证成功后生成 JWT。

2. **服务器签发 JWT**：

服务器将用户 ID、权限等信息放入 **Payload**，使用 **密钥（Secret）** 进行签名，生成 Token，并返回给客户端。

3. **客户端存储 JWT**：

客户端可将 JWT 存储在 **localStorage、sessionStorage** 或 **HttpOnly Cookie** 中。

4. **客户端请求 API**：

在请求头中携带 JWT，例如：

```json
Authorization: Bearer <token>
```



### JWT优/缺点

- 优点

✅ **无状态**：无需在服务器端存储会话数据，适用于微服务架构。

✅ **跨平台**：JSON 格式易解析，可用于不同编程语言。

✅ **安全**：通过签名防篡改，结合 HTTPS 传输更安全。

- 缺点

> **❌Token 体积较大**
>
> **❌ 无法撤销（Revoke）或修改**
>
> ​	•	由于 JWT 是 **无状态的**，一旦颁发，就无法单独撤销某个 Token，除非使用黑名单或缩短 Token 过期时间。
>
> ​	•	相比传统的 Session 机制，Session 可以随时在服务器端销毁，而 JWT 只能等到过期才自动失效。
>
> 
>
> ❌ **存储方式有安全风险**
>
> ​	•	如果将 JWT 存储在 **localStorage**，可能会受到 **XSS（跨站脚本攻击）** 的威胁。
>
> ​	•	如果存储在 **HttpOnly Cookie**，则可以防止 XSS 攻击，但仍然可能受到 **CSRF（跨站请求伪造）** 的影响，需要额外的防护措施。
>
> 
>
> **❌安全风险**
>
> ​	•	**签名算法漏洞**：如果使用 **HS256** 并且密钥较弱，攻击者可能会暴力破解密钥。建议使用 **RS256（非对称加密）** 提高安全性。
>
> ​	•	**无加密**：JWT **默认不会加密**，Payload 里的数据可以被解码，所以敏感信息（如密码）不应存储在 JWT 内。
>
> ​	•	**过期 Token 重放攻击**：如果 Token 过期后，服务器没有校验 **iat（签发时间）**，可能会被攻击者利用。
>
> **❌增加服务器负担**
>
> ​	•	每次请求都需要在服务器端进行 **签名校验（Signature Validation）**，相比 Session ID 直接查数据库或 Redis 可能更消耗 CPU 资源。

- 解决办法：

1. **短生命周期的 JWT + Refresh Token 机制**（如果 JWT 过期，服务器返回 **401 Unauthorized**。客户端/前端定期发送refresh token请求）。
2. **Token 黑名单**（在 Redis 或数据库存储已注销的 Token）。
3. **结合 Session 机制**（使用 JWT 进行身份认证，Session 存储权限信息）。



## tcp粘包

- 固定长度消息

- 特殊字符作为边界

  <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250130%E4%B8%8A%E5%8D%8810308714.png" alt="image-20250130上午10308714" style="zoom:50%;" />

- 自定义消息结构

  例如：

  ```c
  struct { 
      u_int32_t message_length; 
      char message_data[]; 
  } message;
  ```

  
