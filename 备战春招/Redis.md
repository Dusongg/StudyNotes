

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





# 缓存与数据库一致性

- 缓存更新策略？
- 旁路缓存策略实现？
- 如何保证两个操作（删缓存 + 改数据库）的原子性？：1️⃣消息队列2️⃣订阅binlog3️⃣TCC





# 大key & 热key





# 持久化策略

- RDB
- AOF
- 混合持久化
  - 效果：结合 RDB 的高效恢复能力和 AOF 的数据完整性
  - 原理：
    - 混合持久化工作在 **AOF 日志重写（`bgrewriteaof`）过程**，当开启了混合持久化时（`aof-use-rdb-preamble yes`）
    - 在 AOF 重写日志时，fork 出来的重写子进程会先将与主线程共享的内存数据以 RDB 方式写入到 AOF 文件，
    - 然后主线程处理的操作命令会被记录在重写缓冲区里，重写缓冲区里的增量命令会以 AOF 方式写入到 AOF 文件




# 其他

## 本地缓存与Redis缓存的区别





