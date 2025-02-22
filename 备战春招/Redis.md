

# 过期删除与内存淘汰策略

1. 过期删除：惰性删除 + 定期删除（**每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。**）

- 目的：权衡使用 CPU 时间和内存浪费

2. 内存淘汰：范围分在<u>所有key中</u> 与 <u>在设置过期时间的key中</u>
   - LFU ：Redis 实现的是一种**近似 LRU 算法**，目的是为了更好的节约内存，它的**实现方式是在 Redis 的对象结构体中添加一个额外的字段，用于记录此数据的最后一次访问时间**。
   
     当 Redis 进行内存淘汰时，会使用**随机采样的方式来淘汰数据**，它是随机取 5 个值（此值可配置），然后**淘汰最久没有使用的那个**。（**有缓存污染的问题**）
   
   - LRU：最近最不常用，记录访问频率



#  缓存穿透/雪崩/击穿

1. 缓存穿透

![image-20240906154038533](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906154038533.png)

- 其他方法：热点参数限流、增强id的复杂度。。。

2. 缓存雪崩

![image-20240906160036211](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906160036211.png)

3.  缓存击穿

![image-20240906160743688](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906160743688.png)

- 逻辑过期：不设置TTL, 只在Value中添加一个过期时间

![image-20240906161241957](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906161241957.png)

### go通过singleflight包解决

- 只有第一个go程才能进入临界区访问mysql并更新缓存，其他go程等待第一个的结果（通过`wg.Wait`唤醒）
- 原理与互斥锁的原理类似

![image-20241003232021825](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241003232021825.png)

- 内部使用WaitGroup实现

```go
type Group struct {
	mu sync.Mutex       // 保护map
	m  map[string]*call // 存储每个 key 的正在执行的 call
}

type call struct {
	wg  sync.WaitGroup // 用于等待任务完成
	val interface{}    // 执行任务的返回值
	err error          // 执行任务的错误
}

//简化版本
func (g *Group) Do(key string, fn func() (interface{}, error)) (interface{}, error, bool) {
	g.mu.Lock()
	if g.m == nil {
		g.m = make(map[string]*call)
	}

	// 如果 key 已存在，表示任务正在进行
	if c, ok := g.m[key]; ok {
		g.mu.Unlock()
		c.wg.Wait() // 等待任务完成
		return c.val, c.err, false
	}

	// key 不存在，创建新的 call
	c := new(call)
	c.wg.Add(1)      // 增加等待计数
	g.m[key] = c     // 将任务加入 map
	g.mu.Unlock()

	// 执行用户提供的函数
	c.val, c.err = fn()
	c.wg.Done()      // 标记任务完成

	// 从 map 中移除任务
	g.mu.Lock()
	delete(g.m, key)
	g.mu.Unlock()

	return c.val, c.err, true
}
```



# [redis分布式锁](../Cloud Native/分布式/03_单机锁与分布式锁.md/#分布式锁)

单节点redis下实现分布式锁需要一下条件：

1. 通过SETNX保证加锁的原子性
2. 加上过期时间 PX/EX
3. 锁的value需要设置为客户端的一个唯一id（删除时先检查在删除，通过lua脚本实现原子性）



问题：

1. 超时时间设置 ——》**看门狗**
2. 主从模式下，由于redis分布式策略偏向**AP**（可用性和分区容错性），选择异步复制的方式（弱一致性），导致分布式锁不可靠 ——》 **redlock**

## Redisson







# 缓存与数据库一致性

- 缓存更新策略？
- 旁路缓存策略实现？
- 如何保证两个操作（删缓存 + 改数据库）的原子性？：1️⃣消息队列2️⃣订阅binlog3️⃣TCC

 



# 大key & 热key





# 持久化策略

## RDB

## AOF

## 混合持久化

- 效果：结合 RDB 的高效恢复能力和 AOF 的数据完整性
- **生成方式**：
  - 当开启混合持久化并触发AOF重写时，Redis会**先以RDB格式生成当前内存的快照**，作为持久化文件的前半部分。
  - 重写期间的新增写操作命令会以**AOF格式**继续追加到文件末尾，形成完整的持久化文件（`appendonly.aof`）。

- **性能影响**：

  - RDB生成期间可能短暂阻塞主线程（尤其在数据量大时），建议在低峰期操作。

  - AOF增量命令的追加对性能影响较小（依赖磁盘写入策略，如`appendfsync`配置为`everysec`平衡性能与安全。



# 数据结构

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240422011248940.png" alt="image-20240422011248940" style="zoom:50%;" />

## String

encoding：int、embstr、raw

1️⃣**embstr和raw的区别**

​	• embstr 是一种优化小字符串（小于44字节）存储的方式，它通过将字符串的内容和元数据存储在一块连续内存区域内，减少了内存分配的开销。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250213%E4%B8%8B%E5%8D%8850125241.png" alt="image-20250213下午50125241" style="zoom:50%;" />

​	• raw 则适用于较大的字符串或需要频繁操作的字符串，虽然它可能占用更多内存，但提供了更大的灵活性。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250213%E4%B8%8B%E5%8D%8850134622.png" alt="image-20250213下午50134622" style="zoom:50%;" />

**2️⃣ 使用场景**

缓存、计数、分布式锁、记录session



## List

### List作为消息队列

- **阻塞读**（list原生保证消息顺序性）：BRPOP命令也称为阻塞式读取，客户端在没有读到队列数据时，自动阻塞，直到有新的数据写入队列，再开始读取新数据

- **重复消息**：全局唯一ID

- **保证消息可靠性**：`BRPOPLPUSH`**消费者程序从一个 List 中读取消息，同时，Redis 会把这个消息再插入到另一个 List（可以叫作备份 List）留存**。

  > 当消费者程序从 List 中读取一条消息后，List 就不会再留存这条消息了。所以，如果消费者程序在处理消息的过程出现了故障或宕机，就会导致消息没有处理完成，那么，消费者程序再次启动后，就没法再次从 List 中读取消息了。

- **List作为消息队列的缺陷**：1️⃣不支持多个消费者消费同一条消息 2️⃣不支持消费组的实现





## Hash

```bash
# 存储一个哈希表uid:1的键值
> HMSET uid:1 name Tom age 15
2
# 存储一个哈希表uid:2的键值
> HMSET uid:2 name Jerry age 13
2
# 获取哈希表用户id为1中所有的键值
> HGETALL uid:1
1) "name"
2) "Tom"
3) "age"
4) "15"
```

应用场景



## Set

应用场景








# 其他

## 本地缓存与Redis缓存的区别

- redis实现多种数据结构，优化内存存储
- redis基于内存操作对外提供kv操作，可 处理多个主机的请求
- 在分布式场景下redis实现了高可用、可拓展





# redis实现消息队列的方式

- [rabbitMQ](../RabbitMQ.md)

**1️⃣ list实现**

LPUSH + BRPOP 实现

> - 单消费者

2️⃣**发布订阅实现**

![image-20240921203124380](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921203124380.png)

> 不支持数据持久化
>
> 无法保证可靠性
>
> 消息堆积有上限，超出时数据丢失（消费者无法接收到离线时的数据）

**3️⃣[Streams](https://redis.io/docs/latest/commands/?group=stream)**

**支持消息的持久化、支持自动生成全局唯一 ID、支持 ack 确认消息的模式、支持消费组模式等，让消息队列更加的稳定和可靠**

- **即同一个消费组里的消费者不能消费同一条消息**。不同消费者组可以重复消费

  



1. `XADD`生产

![image-20240921204204306](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921204204306.png)

2. `XREAD`消费

通过`XREAD`的`$`读取，可能会出现==消息漏读==的问题，因为`$`只会读取最新消息

![image-20240921204701258](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921204701258.png)

3. `XGROUP`消费者组

![image-20240921205309401](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921205309401.png)

- `XGROUP CREATE`
- `XREADGROUP`：**消费者可以在重启后，用 XPENDING 命令查看已读取、但尚未确认处理完成的消息**。



## Redis 基于 Stream 消息队列与专业的消息队列有哪些差距？

- 消息丢失情况

  - AOF 持久化配置为每秒写盘，但这个写盘过程是异步的，Redis 宕机时会存在数据丢失的可能

  - 主从复制也是异步的，[主从切换时，也存在丢失数据的可能 (opens new window)](https://xiaolincoding.com/redis/cluster/master_slave_replication.html#redis-主从切换如何减少数据丢失)。

- 消息堆积情况

> 所以 Redis 的 Stream 提供了可以指定队列最大长度的功能，就是为了避免这种情况发生。
>
> 当指定队列最大长度时，队列长度超过上限后，旧消息会被删除，只保留固定长度的新消息。这么来看，Stream 在消息积压时，如果指定了最大长度，还是有可能丢失消息的。
>
> 但 Kafka、RabbitMQ 专业的消息队列它们的数据都是存储在磁盘上，当消息积压时，无非就是多占用一些磁盘空间。





## Redis 发布/订阅机制为什么不可以作为消息队列？

发布订阅机制存在以下缺点，都是跟丢失数据有关：

1. 发布/订阅机制没有基于任何数据类型实现，所以不具备「数据持久化」的能力，也就是发布/订阅机制的相关操作，不会写入到 RDB 和 AOF 中，当 Redis 宕机重启，发布/订阅机制的数据也会全部丢失。
2. 发布订阅模式是“发后既忘”的工作模式，如果有订阅者离线重连之后不能消费之前的历史消息。
3. 当消费端有一定的消息积压时，也就是生产者发送的消息，消费者消费不过来时，如果超过 32M 或者是 60s 内持续保持在 8M 以上，消费端会被强行断开，这个参数是在配置文件中设置的，默认值是 `client-output-buffer-limit pubsub 32mb 8mb 60`。 



## 数据库服务器为什么适合开多个线程，而redis一个线程就很快呢

✅ **数据库服务器**使用**多线程**，因为它需要：

​	•处理大量并发请求

​	•进行复杂的事务和查询优化

​	•频繁访问磁盘 I/O

✅ **Redis**主要使用**单线程**，因为：

​	•主要在内存中操作数据，避免 I/O 瓶颈

​	•采用 I/O 多路复用，高效处理并发

​	•避免锁竞争，减少线程切换的开销

但是在 Redis 6.0 之后，它也开始引入**多线程**来优化网络 I/O，进一步提高并发能力。

## 什么场景下不适合用Redis

-  **数据量超大且无法装入内存的场景**：

-  **需要强事务支持的场景**：

  虽然Redis支持事务（通过MULTI/EXEC命令），但它并不提供与关系型数据库一样强大的事务机制（如ACID事务）

- **需要强一致性的分布式场景**：

Redis采用的是最终一致性模型（如Redis Cluster），这意味着它在某些情况下可能会出现数据不一致的情况。如果应用需要强一致性的分布式特性，可能需要使用支持强一致性的分布式数据库（如Paxos协议的Zookeeper、CockroachDB等）。





