#  计网基础

## DNS（Domain Name System）解析步骤 

- 默认端口号是53

1️⃣查看浏览器缓存

2️⃣ 查看操作系统缓存

3️⃣查看host文件

4️⃣ 询问本地DNS服务器（服务器本地缓存 —> 根域名服务器(.) —> 顶级域名服务器(.com) —> 权威DNS服务器(server.com)）

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250215%E4%B8%8B%E5%8D%8840330956.png" alt="image-20250215下午40330956" style="zoom:50%;" />

#### 为什么DNS解析需要这么复杂

1️⃣ **分散负载**：

​	•<u>如果所有DNS查询都直接由一个单一的服务器来处理，这会导致该服务器的负载过高，响应速度变慢，甚至崩溃。</u>通过将DNS解析任务分布到多个层级的服务器（根DNS服务器、顶级域DNS服务器、权威DNS服务器等），可以有效分散请求负载，提高整个系统的稳定性和响应速度。



2️⃣ **去中心化**：

​	•分层的DNS架构使得<u>整个系统没有单点故障</u>，每个域名的解析任务可以由不同的权威DNS服务器独立处理。这样即使某个权威DNS服务器或某个域名解析出现问题，其他域名的解析不会受到影响。



3️⃣ **提高扩展性**：

​	•DNS的分层结构使得系统非常易于扩展。<u>每个DNS服务器负责管理一部分域名，并可以独立地进行扩展和更新</u>。例如，根DNS服务器不需要知道每个具体域名的解析，而是通过与顶级域和权威DNS服务器的合作来实现扩展。

## MSS、MTU相关

- `MTU`：一个网络包的最大长度，以太网中一般为 `1500` 字节。
- `MSS`：除去 IP 和 TCP 头部之后，一个网络包所能容纳的 TCP 数据的最大长度。

![数据包分割](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/12.jpg)

### 为什么有了MTU还需要有MSS

**MTU 作用于链路层**，决定单帧最大大小，包括 TCP/IP 头和数据。

**MSS 作用于 TCP 层**，决定单个 TCP 段的最大数据负载，以避免 IP 分片，提高传输效率。

- 作用：1️⃣防止IP分片，提高传输效率， 2️⃣适应不同的网络环境

## 内核接受网络包流程

### NAPI（New API）

通过**减少中断频率**，结合**中断驱动和轮询机制**，提高网络数据包的处理效率并减少系统资源的开销。

- 传统网络终端弊端：<u>”中断风暴“</u>
- NAPI优化：在接收到数据包时，最初仍通过中断方式通知内核。进入内核后，<u>中断被禁用</u>，改用<u>轮询方式批量处理数据包</u>（软中断）。

### 网络协议栈解析处理

- **传输层—>socket层**：根据四元组`{源 IP、源端口、目的 IP、目的端口}` 作为标识，找出对应的 Socket，并把数据放到 Socket 的接收缓冲区

- 应用层程序调用 Socket 接口，将内核的 <u>Socket 接收缓冲区的数据「拷贝」到应用层的缓冲区</u>，然后唤醒用户进程。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E6%94%B6%E5%8F%91%E6%B5%81%E7%A8%8B.png)

### [Linux发送网络包流程](https://xiaolincoding.com/network/1_base/how_os_deal_network_package.html#linux-%E5%8F%91%E9%80%81%E7%BD%91%E7%BB%9C%E5%8C%85%E7%9A%84%E6%B5%81%E7%A8%8B)

如上图右侧所示

> 首先，应用程序会调用 Socket 发送数据包的接口，由于这个是系统调用，所以会从用户态陷入到内核态中的 Socket 层，内核会申请一个内核态的 sk_buff 内存，**将用户待发送的数据拷贝到 sk_buff 内存，并将其加入到发送缓冲区**。
>
> 接下来，网络协议栈从 Socket 发送缓冲区中取出 sk_buff，并按照 TCP/IP 协议栈从上到下逐层处理。
>
> 如果使用的是 TCP 传输协议发送数据，那么**先拷贝一个新的 sk_buff 副本** ，这是因为 sk_buff 后续在调用网络层，最后到达网卡发送完成的时候，这个 sk_buff 会被释放掉。而 TCP 协议是支持丢失**重传**的，在收到对方的 ACK 之前，这个 sk_buff 不能被删除。所以内核的做法就是每次调用网卡发送的时候，实际上传递出去的是 sk_buff 的一个拷贝，等收到 ACK 再真正删除





## 打开网页的过程

- 解析URL：分析 URL 所需要使用的**传输协议和请求的资源路径**。如果输入的 URL 中的协议或者主机名不合法，将会把地址栏中输入的内容传递给搜索引擎。如果没有问题，浏览器会检查 URL 中是否出现了非法字符，则对非法字符进行转义后在进行下一过程。
- 缓存判断：浏览器会判断所请求的资源是否在缓存里，如果请求的资源在缓存里且没有失效，那么就直接使用，否则向服务器发起新的请求。
- DNS解析：如果资源不在本地缓存，首先需要进行DNS解析。浏览器会向本地DNS服务器发送域名解析请求，本地DNS服务器会逐级查询，最终找到对应的IP地址。
- 获取MAC地址：当浏览器得到 IP 地址后，数据传输还需要知道目的主机 MAC 地址，因为应用层下发数据给传输层，TCP 协议会指定源端口号和目的端口号，然后下发给网络层。网络层会将本机地址作为源地址，获取的 IP 地址作为目的地址。然后将下发给数据链路层，数据链路层的发送需要加入通信双方的 MAC 地址，本机的 MAC 地址作为源 MAC 地址，目的 MAC 地址需要分情况处理。通过将 IP 地址与本机的子网掩码相结合，可以判断是否与请求主机在同一个子网里，如果在同一个子网里，可以使用 APR 协议获取到目的主机的 MAC 地址，如果不在一个子网里，那么请求应该转发给网关，由它代为转发，此时同样可以通过 ARP 协议来获取网关的 MAC 地址，此时目的主机的 MAC 地址应该为网关的地址。
- 建立TCP连接：主机将使用目标 IP地址和目标MAC地址发送一个TCP SYN包，请求建立一个TCP连接，然后交给路由器转发，等路由器转到目标服务器后，服务器回复一个SYN-ACK包，确认连接请求。然后，主机发送一个ACK包，确认已收到服务器的确认，然后 TCP 连接建立完成。
- HTTPS 的 TLS 四次握手：如果使用的是 HTTPS 协议，在通信前还存在 TLS 的四次握手。
- 发送HTTP请求：连接建立后，浏览器会向服务器发送HTTP请求。请求中包含了用户需要获取的资源的信息，例如网页的URL、请求方法（GET、POST等）等。
- 服务器处理请求并返回响应：服务器收到请求后，会根据请求的内容进行相应的处理。例如，如果是请求网页，服务器会读取相应的网页文件，并生成HTTP响应。



## 点到点和端到端的区别

点到点（Point-to-Point, P2P）和端到端（End-to-End, E2E）的主要区别如下：

1️⃣ **定义**

​	•	**点到点（P2P）**：指的是通信或数据传输在两个特定节点之间进行，通常是单一的直接连接。例如，两台计算机之间通过专用链路通信。

​	•	**端到端（E2E）**：指的是通信或数据传输从发送端一直到接收端，中间可能经过多个中继或网络设备，但最终确保端点之间的完整性。例如，互联网中的TCP/IP协议保障数据从源头设备到目标设备的完整传输。

2️⃣ **中间节点的作用**

​	•	**P2P**：通常只涉及两个节点，可能不需要额外的网络设备（如路由器、网关）来完成通信。

​	•	**E2E**：数据可能经过多个路由器、交换机等网络设备，但最终保证数据的完整交付。

3️⃣ **应用场景**

​	•	**P2P**：文件传输（如FTP）、串口通信、两点间加密通信（如VPN隧道）。

​	•	**E2E**：互联网应用（如HTTPS、VoIP、视频会议）、端到端加密（如E2EE消息加密）。

4️⃣ **可靠性和控制**

​	•	**P2P**：通常由通信双方直接控制，适用于高可靠性、低延迟的连接。

​	•	**E2E**：需要协议（如TCP）确保数据完整性，可能受网络中间设备的影响，如丢包、延迟等。

总结来说，**点到点主要关注的是直接通信连接，而端到端则是保证从发送端到接收端的完整传输，即使数据经过多个中间节点。**

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
-   **401 Unauthorized**
- 「**403 Forbidden**」表示服务器禁止访问资源，并不是客户端的请求出错。
- 「**404 Not Found**」表示请求的资源在服务器上不存在或未找到，所以无法提供给客户端。

`5xx` 类状态码表示客户端请求报文正确，但是**服务器处理时内部发生了错误**，属于服务器端的错误码。

- 「**501 Not Implemented**」表示客户端请求的功能还不支持，类似“即将开业，敬请期待”的意思。
- 「**502 Bad Gateway**」通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问后端服务器发生了错误。
- 「**503 Service Unavailable**」表示服务器当前很忙，暂时无法响应客户端，类似“网络服务正忙，请稍后重试”的意思。
- **504 Gateway Time-out**：作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器收到响应。



### 常见字段

1️⃣ 身份认证

**Authorization**：客户端认证信息（如 Bearer <token>）

**Cookie**：携带客户端的 Cookie（如 Cookie: sessionId=abc123）

**Set-Cookie**：服务器设置 Cookie（如 Set-Cookie: sessionId=xyz456; HttpOnly）

2️⃣ 缓存相关

强制缓存：Cache-Control、Expires

协商缓存1:If-None-Match、ETag

协商缓存2:If-Modified-Since、Last-Modified

**3️⃣ 内容协商相关**

**Accept**：客户端可接受的响应格式（如 Accept: application/json）

**Accept-Encoding**：客户端支持的编码格式（如 gzip, deflate）

**Accept-Language**：客户端首选语言（如 zh-CN, en-US）

**Content-Type**：请求/响应的内容类型（如 application/json、text/html）

**Content-Encoding**：响应内容的压缩方式（如 gzip）

**Content-Language**：响应的语言（如 zh-CN）

4️⃣其他

**Location**：重定向地址（如 Location: https://example.com）

**Host**：请求的服务器主机（如 Host: www.example.com）

**User-Agent**：客户端的身份信息（如 Mozilla/5.0）

**Connection**：控制是否保持 TCP 连接（如 keep-alive）

**Upgrade**：用于协议升级（如 Upgrade: websocket）

## HTTP缓存

是的，你的分类是合理的，**强制缓存** 和 **协商缓存** 都是 HTTP 缓存机制的重要组成部分，具体说明如下：

**2️⃣ 缓存相关**

**🔹 强制缓存**

强制缓存可以让浏览器在**缓存有效期内**直接使用本地缓存，不请求服务器。

​	•	**Cache-Control**：控制缓存行为（如 max-age=3600 表示缓存 1 小时，no-cache 表示必须重新验证缓存）。

​	•	**Expires**：设置资源的过期时间（如 Expires: Wed, 21 Oct 2025 07:28:00 GMT）。

​	**说明**：Cache-Control 的优先级高于 Expires，==Expires 依赖于**本地时间**，如果客户端时间不准确可能导致缓存失效问题。==

**🔹 协商缓存**

协商缓存用于在强制缓存失效后，由浏览器向服务器发送请求，服务器根据资源是否更新来决定是否返回新资源。

- **方式 1️⃣ (ETag & If-None-Match)**

​	•	**ETag**：服务器为资源生成的唯一标识（类似哈希值，如 ETag: "abc123"）。

​	•	**If-None-Match**：客户端请求时带上上次获取的 ETag，服务器检查是否匹配。

​	•	**匹配** ➝ 返回 <u>304 Not Modified</u>（使用本地缓存）。

​	•	**不匹配** ➝ 服务器返回新的资源和新的 ETag 值。

- **方式 2️⃣ (Last-Modified & If-Modified-Since)**

​	•	**Last-Modified**：资源的最后修改时间（如 Last-Modified: Wed, 21 Oct 2023 07:28:00 GMT）。

​	•	**If-Modified-Since**：客户端请求时带上 Last-Modified 值，服务器检查资源是否被修改。

​	•	**未修改** ➝ 返回 <u>304 Not Modified</u>（使用本地缓存）。

​	•	**已修改** ➝ 服务器返回新的资源和新的 Last-Modified 值。

- **ETag 与 Last-Modified 的区别**：

​	•ETag 更精确，它是资源的唯一标识，即使资源内容发生微小变化也会变化。

​	•Last-Modified 只能精确到**秒级**，如果资源在 1 秒内多次更新，无法检测出变化。

​	•一般 ETag 优先级**高于** Last-Modified，服务器通常会**同时返回**两者。

🚀 这样分类后，缓存的工作原理和优先级也更加清晰！

![image-20250113下午115124506](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250113%E4%B8%8B%E5%8D%88115124506.png)



## HTTPS

### 数字证书颁发和验证

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250114%E4%B8%8A%E5%8D%88120830275.png" alt="image-20250114上午120830275" style="zoom:50%;" />

- 首先 CA 会把持有者的公钥、用途、颁发者、有效时间等信息打成一个包，然后对这些信息进行 Hash 计算，得到一个 Hash 值；
- 然后 CA 会使用自己的私钥将该 Hash 值加密，生成 Certificate Signature，也就是 CA 对证书做了签名；
- 最后将 Certificate Signature 添加在文件证书上，形成数字证书；



### 🌟基于TLS1.2的RSA握手

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





### 🌟基于TLS1.2的ECDHE握手

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4@main/%E7%BD%91%E7%BB%9C/https/ech_tls%E6%8F%A1%E6%89%8B.png)

ECDHE（椭圆曲线 Diffie-Hellman Ephemeral）是一种基于椭圆曲线的临时密钥交换协议，广泛用于 TLS（如 TLS 1.2 和 TLS 1.3）等加密通信中，以实现**前向安全性（Forward Secrecy）**。以下是 ECDHE 握手过程的详细步骤：

**1️⃣ 客户端发起握手（ClientHello）**	

主要包含：支持的 **TLS 版本**、支持的 **密码套件（Cipher Suites）**（包括 ECDHE 相关的套件，如 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256）、支持的 **椭圆曲线（Elliptic Curves）**、生成的 **随机数（Client Random）**

**2️⃣ 服务器响应（ServerHello & ==Server Key Exchange== & Certificate）**

- **ServerHello**

选择的 **TLS 版本**、选定的 **密码套件**、服务器生成的 **随机数（Server Random）**

-  ==**Server Key Exchange**==（RSA握手没有这一步）

​	服务器选择的椭圆曲线参数、服务器的 **公钥**$ Q_s = d_s G$ （其中 $d_s$ 是私钥， G 是基点）

- **Certificate**

​	使用服务器私钥对该公钥进行签名，确保公钥的真实性（若使用 RSA 证书，服务器私钥签名整个 ServerKeyExchange））

**3️⃣ Client Key Exchange  & Change Cipher Spec & Handshake**

​	计算 **共享密钥**：$S = d_c Q_s = d_c d_s G$

由于只有客户端和服务器知道各自的私钥 $d_c$ 和 $d_s$ ，所以只有它们能计算出相同的共享密钥。

客户端将自己的 ECDHE **公钥** $Q_c$ 发送给服务器。

**4️⃣  Change Cipher Spec & Handshake**

服务器收到 $Q_c$ 后，计算共享密钥：$S = d_s Q_c = d_s d_c G$

这与客户端计算的共享密钥一致。

> **Change Cipher Spec**」：告诉对方后续改用对称算法加密通信。
>
> 「**Encrypted Handshake Message**」消息，把之前发送的数据做一个摘要，再用对称密钥加密一下，让对方做个验证，验证下本次生成的对称密钥是否可以正常使用。

### 🌟TLS1.3握手过程

 <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250205%E4%B8%8B%E5%8D%8820142130.png" alt="image-20250205下午20142130" style="zoom:50%;" />

- 客户端在 Client Hello 消息里带上了支持的椭圆曲线，以及这些椭圆曲线对应的公钥。服务端收到后，选定一个椭圆曲线等参数，然后返回消息时，带上服务端这边的公钥。经过这 1 个 RTT，双方手上已经有生成会话密钥的材料了，于是客户端计算出会话密钥，就可以进行应用数据的加密传输了。



📌 **注意**：0-RTT 数据**容易被重放攻击**，因此不能用于敏感操作（如交易、数据库操作）。

**4️⃣ TLS 1.3 vs TLS 1.2 对比**

| **特性**             | **TLS 1.2**                      | **TLS 1.3**                          |
| -------------------- | -------------------------------- | ------------------------------------ |
| **握手时延**         | 2-RTT                            | **1-RTT**（更快）                    |
| **密钥交换**         | RSA、ECDHE、DHE                  | **仅 ECDHE/DHE**                     |
| **前向安全**         | 仅 ECDHE/DHE 具备                | **默认前向安全性**                   |
| **0-RTT 支持**       | ❌ 不支持                         | ✅ **支持**（会话恢复）               |
| **证书泄露影响**     | 服务器私钥泄露后，可解密历史数据 | 服务器私钥泄露**无法解密**历史数据   |
| **证书加密**         | ❌ 服务器证书明文传输             | ✅ **服务器证书加密**                 |
| **密码套件**         | 复杂（支持 CBC、RC4）            | **仅支持 AEAD（AES-GCM、ChaCha20）** |
| **ChangeCipherSpec** | 需要发送                         | **已移除**                           |



**📌 TLS 1.3 主要优化**

✅ **握手更快（1-RTT）**

✅ **默认前向安全性**（移除 RSA 交换，强制 ECDHE）

✅ **更安全的密码套件**（只支持 AEAD，加密更快更安全）

✅ **支持 0-RTT，减少重连开销**（适用于低延迟应用）

✅ **证书加密**（防止中间人嗅探）



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





## 什么是HTTP断点重传/断点续传

HTTP 断点重传（**Resumable Download / Range Request**）是一种支持**从上次中断的地方继续传输数据**的技术，常用于<u>大文件下载，以提高传输效率、减少带宽浪费。</u>

**1️⃣ 断点重传的核心原理**

HTTP **断点续传**主要依赖于 Range 头和 206 Partial Content 响应状态码：

✅ **客户端请求部分数据**（Range 头）

✅ **服务器返回指定范围数据**（206 Partial Content 响应）

✅ **客户端拼接数据，继续下载**

示例：

​	1. **初始请求（完整下载）**

```http
GET /file.zi p HTTP/1.1
Host: example.com
```

​	2. 服务器返回 200 OK，并传输完整文件。

​	3. **断点重传请求（指定起点继续下载）**

```http
GET /file.zip HTTP/1.1
Host: example.com
Range: bytes=500000-  # 从第 500000 字节开始
```

​	•服务器返回 206 Partial Content，并从第 500000 字节开始传输。

**2️⃣ 服务器如何支持断点续传？**

服务器需满足以下条件：

✅ **支持 Range 头**（服务器需解析 Range 并返回相应数据）。

✅ **返回 Accept-Ranges: bytes**  表示服务器支持按字节范围（byte ranges）请求资源，表明可以进行断点续传

✅ **正确返回 206 Partial Content 状态码**。

示例：服务器返回部分内容：

```
HTTP/1.1 206 Partial Content
Accept-Ranges: bytes
Content-Range: bytes 500000-999999/2000000
Content-Length: 1500000
Content-Type: application/zip
```

**3️⃣ 断点重传的常见应用**

🔹 **浏览器下载器**（如 Chrome、Firefox 支持续传）。

🔹 **下载工具**（如 IDM、迅雷、Wget）。

🔹 **流媒体播放**（如在线播放视频可按需加载部分数据）。

🔹 **P2P 文件共享**（分块传输）。

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

- 让历史报文丢弃
- 第四次挥手报文可能丢失，接受对方FIN报文重传

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

#### listen的第二个参数backlog

在 Linux 内核 2.2 之后，backlog 变成 accept 队列，也就是已完成连接建立的队列长度，**所以现在通常认为 backlog 是 accept 队列。**

**但是上限值是内核参数 somaxconn 的大小，也就说 accpet 队列长度 = min(backlog, somaxconn)。**

## TCP的可靠性

### 超时重传与快速重传

**超时重传：**

- 超时重传时间以 `RTO` （Retransmission Timeout 超时重传时间）表示，**超时重传时间 RTO 的值应该略大于报文往返 RTT 的值**
- RTO时间应该是动态变化的，**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送**

**快速重传：**

- 一般方法：三个相同ACK重传，但是不知道改重传一个还是后面的所有

### `SACK`(Selective Acknowledgment)：

  这种方式需要在 TCP 头部「选项」字段里加一个 `SACK` 的东西，它**可以将已收到的数据的信息发送给「发送方」**

  <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250122%E4%B8%8B%E5%8D%8815506456.png" alt="image-20250122下午15506456" style="zoom:50%;" />

### Duplicate SACK(`D-SACK`)

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

    

#### 窗口缩放因子

**📌 窗口缩放因子（Window Scaling Factor）**

窗口缩放因子是 **TCP 窗口缩放选项（Window Scale Option）** 中的一个参数，用于扩大 TCP **接收窗口（RWND）** 的最大值，以适应高带宽延迟（高 BDP）网络。

**1️⃣ 为什么需要窗口缩放？**

​	•	在 **TCP 头部**，窗口大小（Window Size）字段是 **16 位**，最大值只能表示 **65535 字节（64 KB）**。

​	•	对于高带宽、长延迟的网络（如卫星通信、数据中心网络），64 KB 窗口可能 **严重限制 TCP 吞吐量**。

**计算带宽延迟积（BDP）：**

如果带宽 = 100 Mbps，RTT = 100 ms：

但 TCP **默认窗口最多 64 KB**，远远达不到链路能力，导致 TCP 速率受限。

**2️⃣ 窗口缩放因子（Window Scale）解决了什么？**

**RFC 1323** 定义了 **窗口缩放（Window Scaling）**，通过 **窗口缩放因子（Scaling Factor）** 来**扩大接收窗口（RWND）**。

**它的作用是将 TCP 头部中的 Window Size 乘以 2^S（S 为窗口缩放因子）。**

**公式：**$\text{真实窗口大小} = \text{TCP 头部 Window Size} \times 2^S$

其中：

​	•	S 是窗口缩放因子（**取值范围 0~14**）。

​	•	Window Size 是 TCP 头部中的 16 位字段。

​	•	**最大窗口扩展到 1 GB（2^14 × 64 KB = 1 GB）。**

**4️⃣ 窗口缩放的协商机制**

​	•	**窗口缩放选项只能在 TCP 三次握手阶段协商。**

​	•	由 **客户端在 SYN 报文中** 提供 WSopt=S（S 是窗口缩放因子）。

​	•	**如果服务器支持窗口缩放**，则在 **SYN-ACK** 中返回自己的 WSopt 值。

​	•	**如果一方不支持窗口缩放，则 TCP 仍然使用最大 64 KB 的窗口。**

**5️⃣ 窗口缩放的实际应用**

✅ **高带宽、高 RTT 网络（BDP 大）**，如：

​	•	**卫星通信**

​	•	**数据中心长距离传输**

​	•	**5G 网络**

​	•	**CDN（内容分发网络）**

✅ **现代操作系统（Linux、Windows、macOS）默认开启窗口缩放**，以适应大吞吐量网络。

**6️⃣ 总结**

🔹 **窗口缩放因子** 允许 TCP **窗口大小超过 64 KB**，最大可达 **1 GB**，提高高带宽网络的吞吐量。

🔹 **计算公式：**$\text{真实窗口大小} = \text{TCP 头部 Window Size} \times 2^S$

🔹 **必须在 TCP 三次握手阶段协商，不能在连接建立后修改**。

🔹 **现代系统默认开启窗口缩放，支持更大 TCP 吞吐量**。

### 拥塞控制

#### 有了流量控制为什么还要有拥塞控制？

**TCP 流量控制**是一种**基于接收方能力**的机制，用于**防止发送方发送过快，导致接收方处理不过来**，从而避免数据丢失。TCP 通过 **“滑动窗口协议”** 来实现流量控制。

**在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重**



#### 拥塞控制算法

一开始初始化 `cwnd = 1`，表示可以传一个 `MSS` 大小的数据。

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



## TCP vs UDP

### udp一定比tcp快吗？

**1️⃣ UDP 相比 TCP 的优势**

​	✅ **无连接、头部开销小**：UDP 省去了 TCP 的三次握手、连接维护等过程，数据直接发送，理论上更快。

​	✅ **无流量控制、无拥塞控制**：UDP 不会因为丢包或网络拥塞而降低发送速度，适用于低延迟场景，如视频流、实时通信、游戏等。

**2️⃣ TCP 可能比 UDP 更快的情况**

​	❌ **数据可靠性和重传机制**：如果 UDP 发生丢包，应用层需要自己实现可靠性机制（如超时重传），这可能导致额外的开销，甚至比 TCP 还慢。

​	❌ **网络拥塞时**：TCP 拥塞控制（如慢启动）可提高带宽利用率，而 UDP 可能因丢包严重而影响传输效果。

​	❌ **数据量较大时**：TCP 通过滑动窗口和流量控制优化数据传输，而 UDP 可能因丢包和乱序导致额外的处理成本。

### tcp与udp的区别

- 应用场景

- 可靠性

- 头部字段大小

  <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250209%E4%B8%8A%E5%8D%88125842157.png" alt="image-20250209上午125842157" style="zoom: 25%;" />

- 面向链接

- **传输方式**

  - TCP**：面向字节流，数据以流的形式传输，无明确的消息边界。**
  - UDP：面向数据报，以数据报文（Datagram）为单位发送，消息边界明确。



### 如何解决tcp粘包和拆包

#### 什么是粘包和拆包

**1️⃣ 粘包（Packet Sticking）**：**粘包** 指的是 **多个小数据包被合并到一起发送**，提高网络传输效率（Nagle 算法）

**2️⃣ 拆包（Packet Splitting）****拆包** 指的是 **一个数据包被拆分成多个 TCP 报文进行发送**，导致接收方需要多次接收才能获得完整数据。**原因**：发送方 send() 发送的数据超过 TCP **MSS（最大段长度）**，TCP 会拆分数据以适应网络传输。TCP 不是面向消息的协议，无法保证一次 send() 的数据会被完整 recv() 读取。

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


## 

## [tcp思维导图](../笔试面试总结/TCP建立链接前后异常问题.xmind)

# IP

- DHCP客户端端口：68， DHCP服务端端口：67
- DNS端口：53



## ICMP

- ICMP功能：**确认 IP 包是否成功送达目标地址、报告发送过程中 IP 包被废弃的原因和改善网络设置等。**

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8830530839.png" alt="image-20250124下午30530839" style="zoom:50%;" />

- 目标不可达：

  > - 网络不可达代码为 `0`
  > - 主机不可达代码为 `1`
  > - 协议不可达代码为 `2`
  > - 端口不可达代码为 `3`
  > - 需要进行分片但设置了不分片位代码为 `4`：发送端主机发送 IP 数据报时，将 IP 首部的**分片禁止标志位**设置为`1`。根据这个标志位，途中的路由器遇到超过 MTU 大小的数据包时，不会进行分片，而是直接抛弃。

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




## URL和URI的区别

URL（Uniform Resource Locator）和 URI（Uniform Resource Identifier）虽然在某些情况下可以互换使用，但它们的含义和作用是不同的：

**1️⃣ URI（统一资源标识符）**

​	•	**定义**：URI 是一种通用概念，用来唯一标识互联网上的资源。

​	•	**作用**：URI 的作用是标识资源，可以通过 **名称** 或 **位置** 来确定一个资源。

​	•	**分类**：

URI 包括两种类型：

​	•	**URL**：统一资源定位符，表示资源的具体位置（即地址）。

​	•	**URN**：统一资源名称，用来唯一命名资源，而不指明资源的位置。

​	•	**例子**：

​	•	urn:isbn:9780131103627（URN：指一本书，ISBN号）

​	•	http://www.example.com/index.html（URL：表示资源的地址）

​	•	**总结**：URI 是一个更大的概念，URL 和 URN 都是 URI 的子集。

**2️⃣ URL（统一资源定位符）**

​	•	**定义**：URL 是一种特殊的 URI，它不仅标识了资源，还指明了如何访问这个资源的位置（例如协议、主机名、路径等）。

​	•	**作用**：URL 是网络中最常用的形式，用来描述资源的具体位置及其访问方式。

​	•	**组成部分**：

​	•	**协议**：如 HTTP、HTTPS、FTP 等。

​	•	**主机名/域名**：如 www.example.com。

​	•	**端口号**（可选）：如 :8080。

​	•	**路径**：如 /index.html。

​	•	**查询字符串**（可选）：如 ?id=123。

​	•	**片段标识符**（可选）：如 #section2。

​	•	**例子**：

​	•	http://www.example.com/index.html

​	•	ftp://ftp.example.com/file.txt

​	•	**总结**：URL 表示了资源的 **位置** 和 **访问方式**。

**3️⃣ 主要区别**

| **特性**         | **URI**                | **URL**                           |
| ---------------- | ---------------------- | --------------------------------- |
| **含义**         | 标识资源的统一标准     | 标识资源的具体位置                |
| **是否包含位置** | 不一定包含资源的位置   | 必须包含资源的位置                |
| **包含关系**     | URL 是 URI 的一种      | URL 是 URI 的子集                 |
| **例子**         | urn:isbn:9780131103627 | http://www.example.com/index.html |

**4️⃣ 类比帮助理解**



可以把 **URI** 看作一个人的身份证号码，它唯一标识了这个人，而 **URL** 就像这个人的家庭住址，不仅能标识，还能告诉你去哪里找到他。
