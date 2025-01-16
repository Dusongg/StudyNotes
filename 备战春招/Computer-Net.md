# 计网基础

### DNS解析步骤 

1️⃣查看浏览器缓存

2️⃣ 查看操作系统缓存

3️⃣查看host文件

4️⃣ 询问本地DNS服务器（服务器本地缓存 —> 根域名服务器(.) —> 顶级域名服务器(.com) —> 权威DNS服务器(server.com)）

### MSS、MTU相关

- `MTU`：一个网络包的最大长度，以太网中一般为 `1500` 字节。
- `MSS`：除去 IP 和 TCP 头部之后，一个网络包所能容纳的 TCP 数据的最大长度。

![数据包分割](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/12.jpg)

### 内核接受网络包流程

#### NAPI（New API）

通过**减少中断频率**，结合**中断驱动和轮询机制**，提高网络数据包的处理效率并减少系统资源的开销。

- 传统网络终端弊端：”中断风暴“
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
- 「**403 Forbidden**」表示服务器禁止访问资源，并不是客户端的请求出错。
- 「**404 Not Found**」表示请求的资源在服务器上不存在或未找到，所以无法提供给客户端。

`5xx` 类状态码表示客户端请求报文正确，但是**服务器处理时内部发生了错误**，属于服务器端的错误码。

- 「**500 Internal Server Error**」与 400 类型，是个笼统通用的错误码，服务器发生了什么错误，我们并不知道。
- 「**501 Not Implemented**」表示客户端请求的功能还不支持，类似“即将开业，敬请期待”的意思。
- 「**502 Bad Gateway**」通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问后端服务器发生了错误。
- 「**503 Service Unavailable**」表示服务器当前很忙，暂时无法响应客户端，类似“网络服务正忙，请稍后重试”的意思。



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

- 握手时延从`2RTT`降到`1RTT`



#### 证书优化

- **证书传输优化**：对于服务器的证书应该选择椭圆曲线（ECDSA）证书，而不是 RSA 证书，因为在相同安全强度下， ECC 密钥长度比 RSA 短的多。
- 证书验证优化：
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



## RPC与HTTP区别





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
