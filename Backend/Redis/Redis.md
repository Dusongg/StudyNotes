# ==[Redis参考手册](https://redis.io/docs/latest/commands/)==

# 1 基础认识



- NoSQL与SQL

  ![image-20240425234137088](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240425234137088.png)

- 特征

![image-20240425234614363](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240425234614363.png)

## 1.1 安装配置

- 安装：`sudo apt install redis`

- 配置文件`etc/redis/redis.conf`

  ```properties
  bind 0.0.0.0 ::1    #将本地环回改为任意ip
  
  daemonize yes #守护进程启动
  
  requirepass 123123 #设置密码为123123
  
  protected-mode no   #开启后其他主机才能访问
  
  logfile "xxx.log"  #开启日志，并记录到xxx.log中
  ```

  配置完后重启服务器：`service redis-sever restart`

  查看运行状态：`service redis-server status`

  ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240421202357236.png)

- 链接

  ```bash
  redis-cli
  
  #--raw 将二进制编码
  ```

- 退出：`quit` 或 `ctrl + d`

## 1.2 通用命令

```bash
help @generic #查看通用命令文档 
```



- 类型
  - key:字符串
  - value:字符串、哈希表、列表、集合...

1. `get`

```
GET key
```

2. `set` 

```
SET key value
```

3. `exists`

```
EXISTS key [key ...]	#判断多个key，返回key存在  的个数
```

4. `del`

```
DEL key [key ...] #返回删除key的个数
```

5. `expire`：设置过期时间

```
EXPIRE key [seconds]
PEXPIRE key [millisecond]
```

6. `ttl`：(time to live)获取key的过期时间

```
TTL key   #返回-2表示key已经删除了
```

- ==过期策略==

定期删除 与 惰性删除 相结合

7. `type` : 查看key对应的value的值

```redis
redis> SET key1 "value"
"OK"
redis> LPUSH key2 "value"
(integer) 1
redis> SADD key3 "value"
(integer) 1
redis> TYPE key1
"string"
redis> TYPE key2
"list"
redis> TYPE key3
"set"
redis> 
```

8. [`keys`](https://redis.io/docs/latest/commands/keys/)

```
KEYS pattern
```

- pattern

  ![image-20240421224000208](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240421224000208.png)

9. `flushall`：清除所有数据库中的key
9. `scan` ——渐进式遍历

11. `select dbIndex` —— 切换数据库

dbIndex范围为0-15

12. 

## 1.3 数据类型

### 1.3.1 数据结构与内部编码

- raw：原始字符串（C语言中的字符数组）

- embstr：对于短字符串的优化
- ziplist：对于键值对少的时候，通过压缩列表进行优化
- quicklist：从redis3.2开始用quicklist实现list
- skiplist：跳表

![image-20240422011248940](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240422011248940.png)

- 查看实际编码方式

  ```redis
  OBJECT encoding key
  ```

### string

1. `set`

```redis
SET key value [NX | XX] [GET] [EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
```

![image-20240422020234175](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240422020234175.png)

2. `get`

get只能查询字符串类型的value

3. `mget` / `mset`

设置/获取多组键值对

4. `setnx` / `setex` / `psetex`

不存在则设置/另外设置过期时间（单位s）/（单位ms）

5. ```redis
   #将value视为整数，对其+-
   incr		
   incrby
   decr
   decrby
   incrbyfloat		#前面四个只能算整数，incrbyfloat可以算浮点数
   ```

6. | 函数       | 例子                        | 效果                                                  |
   | ---------- | --------------------------- | ----------------------------------------------------- |
   | `append`   | `append key value`          | 追加fvalue                                            |
   | `getrange` | `getrange key start end`    | 截取value的[start, end]区间部分，负数下标表示倒数     |
   | `setrange` | `setrange key offset value` | offset表示从第几个开始                                |
   | `strlen`   | `strlen key`                | 获取value的长度（字符为单位），若不是string类型则报错 |

### key的结构

- 让redis像mysql一样存储
- key：`数据库:表`
- value：`{字段:值, 字段:值...}`

![image-20240426144736318](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240426144736318.png)

### hash

![image-20240426145202610](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240426145202610.png)

### list

![image-20240426150022647](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240426150022647.png)

### set

![image-20240426151000576](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240426151000576.png)

### sorted_set

- 默认按score升序排列，在`Z`后面加上`REV`倒序排列

![image-20240426151322606](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240426151322606.png)



## 1.4 单线程模型

redis虽然是单线程模型，为什么效率却这么高

1. 访问内存，MySQL则多是访问硬盘
2. 核心功能简单
3. 单线程模型，避免了多线程带来的开销
4. 底层使用epoll





# 2 redis客户端

- redis自定义应用层协议

## 2.1 RESP协议（Redis serialization protocol）

# 3 分布式缓存 —— redis集群

## 3.1 持久化

- 单节点Redis的问题：1. 数据丢失  2. 并发能力  3. 存储能力  4. 故障恢复

### 3.1.1 RDB

![image-20240427230804132](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240427230804132.png)

![image-20240427233636584](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240427233636584.png)

### 3.1.2 RDB底层原理

![image-20240427234336347](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240427234336347.png)





### 3.1.3 AOF持久化

![image-20240427235256612](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240427235256612.png)

- 默认选择`appendfsync everysec`

![image-20240427235540933](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240427235540933.png)

- 解决AOF文件过大的问题（清除无效数据） —— 执行`bgrewriteaof`命令

![image-20240428000342657](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428000342657.png)



### 3.1.4 AOF与RDB的对比

![image-20240428200406918](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428200406918.png)



##  3.2 Redis主从

### 3.2.1 搭建主从集群

（转Redis集群.md）

### 3.2.2 数据同步原理

1. 全量同步

- <u>**id不同-  >  生成RDB做全量同步**</u>
- **<u> id相同  ->  通过offset做增量同步</u>**

![image-20240428202606843](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428202606843.png)

![image-20240428202728682](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428202728682.png)



2. 增量同步

![11111](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428203155649.png)

- ==如何优化主从集群同步==

![image-20240428203425431](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428203425431.png)

## 3.3 Redis哨兵

![image-20240428205054062](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428205054062.png)

#### Sentinel监控原理

Sentinel基于心跳机制监测服务状态，每隔1秒向集群的每个实例发送ping命令

==主观下线==：如果某sentinel节点发现某实例<u>未在规定时间响应</u>，则认为该实例主观下线。

==客观下线==：若超过指定数量（quorum)的sentinel都认为该实例主观下线，则该实例客观下线。quorum值最好超过Sentinel实例数量的一半。

#### 选择新的master

![image-20240428205910199](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428205910199.png)

#### 如何实现故障转移

![image-20240428210025947](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428210025947.png)

### 3.3.1 搭建哨兵集群

（转Redis集群.md）

###  3.3.2 Redis客户端状态感知

 

## 3.4 Redis分片集群

### 3.4.1 搭建分片集群

（转Redis集群.md）

![image-20240428223237874](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428223237874.png)

###  3.4.2 散列插槽

![image-20240428225102178](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428225102178.png)

- 如何将同一类数据固定的保存在同一个Redis实例中

![image-20240428225417930](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428225417930.png)

### 3.4.3 集群伸缩

- 创建新Redis，加入到集群中

![image-20240428225728626](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428225728626.png)

- 分配插槽

```SH
redis-cli --cluster reshard ip:port
```

### 3.4.4 故障转移

1. 自动转移

- 一个服务突然宕机，slave节点自动转移成master

2. 手动转移

![image-20240428231148079](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428231148079.png)



# 4 多级缓存

![image-20240428232744878](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240428232744878.png)



##  4.1 JVM进程缓存

 

## 4.2 OpenResty

![image-20240429231721675](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240429231721675.png)

![image-20240429234315794](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240429234315794.png)

1. 修改nginx.conf文件

![image-20240429235323238](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240429235323238.png)

2. 编写item.lua文件

![image-20240429235435025](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240429235435025.png)

- 获取请求参数

![image-20240429235839151](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240429235839151.png)

- 封装http请求，由OpenResty发送给后端

![image-20240430001548649](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240430001548649.png)

![image-20240430001527599](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240430001527599.png)



- 使用common.lua，通过item.lua发送请求到后端

![image-20240430003124528](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240430003124528.png)



- Redis预热





- OpenResty访问Redis缓存

![image-20240501150214696](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240501150214696.png)



- nginx本地缓存

## 4.3 缓存同步

![image-20240502121041244](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502121041244.png)

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502121041244.png)

### 4.3.1 Canel

![image-20240502121331454](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502121331454.png)

![image-20240502220556739](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502220556739.png)

# 5 Redis实践优化

## 5.1 key的设计

![image-20240502221558171](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502221558171.png)

- 什么是Big Key

单个key的value小于10KB

对于集合类型的key，建议元素数量小于1000

- Big Key的危害

![image-20240502222457055](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502222457055.png)

- 删除Big Key

![image-20240502223603980](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240502223603980.png)



## 5.2 批处理优化

- 批处理一次性执行太多任务会阻塞网络

1. 原生的M操作
2. Pipeline

- pipeline的多个命令不具有原子性

![image-20240503115012875](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240503115012875.png)

### 5.2.1 集群模式下的批处理

![image-20240503120412597](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240503120412597.png)

## 5.3 服务端优化

### 5.3.1 持久化配置

![image-20240503223951603](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240503223951603.png)

### 5.3.2 慢查询

- 该配置重启会恢复，要用就配置可以修改配置文件

- 其他命令

![image-20240504000236684](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504000236684.png)

### 5.3.3 服务器安全配置

![image-20240504135349697](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504135349697.png)

- 安全配置

1. Redis一定要设置密码
2. 禁止线上使用下面命令: keys、flushall、flushdb、config set等命令。可以利用rename-command禁用。（在redis.conf文件下设置  `rename-command keys xxx123xxxdfsda`）
3. bind:限制网卡，禁止外网网卡访问禁止线上使用下面命令（在redis.conf文件下设置）
4. 开启防火墙开启防火墙
5. 不要使用Root账户启动Redis
6. 尽量不要使用默认端口

### 5.3.4 内存配置

![image-20240504140451848](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504140451848.png)

- 查看内存使用情况的命令：
  - `memory xxx`
  - `info memory`

![image-20240504142118346](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504142118346.png)

- 两个查看客户端链接信息的命令
  - `info client`
  - `client list`

### 5.3.5 集群相关问题

- 集群的完整性问题

![image-20240504143153125](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504143153125.png)

- 集群的带宽问题

![image-20240504143434456](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504143434456.png)

- 集群带来的其他问题

![image-20240504143734732](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504143734732.png)





# 6 Redis底层原理

## 6.1 底层数据结构

### 6.1.1 SDS动态数组

![image-20240504150555378](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504150555378.png)

![image-20240504150612583](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504150612583.png)



### 6.1.2 Intset

![image-20240504150909130](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504150909130.png)

- intset升级

![image-20240504152243964](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504152243964.png)

- 优点

1. Redis会确保Intset中的元素唯一、有序
2. 具备类型升级机制，可以节省内存空间
3. 底层采用二分查找方式来查询

### 6.1.3 Dict

- Dict由三部分组成：

![image-20240504153208343](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504153208343.png)

![image-20240504153407129](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504153407129.png)

- Dict的扩容机制

![image-20240504163735702](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504163735702.png)

- 渐进式rehash

![ss](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504165301976.png)

### 6.1.4 ZipList

![image-20240504165810053](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504165810053.png)

![image-20240504170359861](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504170359861.png)

- encoding编码（小端）

  - 字符串类型编码（00、01、10开头）

    ![image-20240504171221619](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504171221619.png)

  - 整数类型编码

    ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504171933002.png)

- 连锁更新问题

![image-20240504230957638](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504230957638.png)

### 6.1.5 QuickList

- QuickList = LinkList + ZipList

- QuickList：限制ZipList的大小，解决内存分配的效率问题

![image-20240504233808521](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504233808521.png)

- 如何限制ZipList大小

![image-20240504233856667](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504233856667.png)

- QuickList节点压缩![image-20240504234159575](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504234159575.png)

![image-20240504235503333](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504235503333.png)

![image-20240504235643390](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240504235643390.png)

### 6.1.5 SkipList

![image-20240505000844632](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240505000844632.png)

- 底层结构

![image-20240505001646311](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240505001646311.png)

![image-20240505001031349](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240505001031349.png)



### 6.1.6 RedisObject

![image-20240507103139134](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507103139134.png)

- 11种编码格式

![image-20240507103439069](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507103439069.png)

- 数据类型对应的编码格式

![image-20240507103733752](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507103733752.png)

## 6.2 五种数据结构

### 6.2.1 String

- `raw`：其基本编码方式是RAW，基于简单动态字符串(SDS）实现，存储上限为512mb。
- `embstr`：如果存储的SDS长度小于<u>44字节</u>，则会采用EMBSTR编码，此时object head与SDS是一段连续空间。申请内存时只需要调用一次内存分配函数，效率更高。
- `int`：如果存储的字符串是整数值，并且大小在LONG_MAX范围内，则会采用INT编码:直接将数据保存在RedisObject的ptr指针位置（刚好8字节)，不再需要SDS了。

![image-20240507105438669](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507105438669.png)

### 6.2.2 List

![image-20240507110929648](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507110929648.png)



### 6.2.3 Set

Set是Redis中的集合，不一定确保元素有序，可以满足元素唯一、查询效率要求极高。

- `Dict`：为了查询效率和唯一性，set采用HT编码（Dict)。Dict中的key用来存储元素, value统一为null。

- `IIntSet`：当存储的所有数据都是整数，并且元素数量不超过`set-max-intset-entries`时，Set会采用IntSet编码，以节省内存。

![image-20240507112417630](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507112417630.png)

- 插入时类型转换

![image-20240507112431813](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507112431813.png)

### 6.2.4 ZSet

zset底层数据结构必须满足键值存储、键必须唯一、可排序这几个需求

- SkipList:可以排序，并且可以同时存储score和ele值（ member)
- HT ( Dict):可以键值存储，并且可以根据key找value

![image-20240507153935285](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507153935285.png)

- 当元素不多时，使用ZipList编码

![image-20240507154502329](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507154502329.png)

![image-20240507154916702](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507154916702.png)

### 6.2.5 Hash

![image-20240507155424965](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507155424965.png)

## 6.3 Redis网络模型

### 6.3.1 select

![image-20240507203420468](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507203420468.png)



### 6.3.2 poll

![image-20240507203831453](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507203831453.png)



### 6.3.3 epoll

![image-20240507205127392](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507205127392.png)



### 6.3.4 Redis网络模型

单线程？多线程？ 

![image-20240507212223799](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507212223799.png)

![image-20240507222739878](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240507222739878.png)



## 6.4 Redis通信协议

![image-20240510101118195](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240510101118195.png)









# 7 Redis实战

## 7.1 短信登录

![image-20240905170425429](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240905170425429.png)

![image-20240905170443544](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240905170443544.png)



## 7.2 商户查询缓存

### 7.2.1 缓存更新策略

![image-20240906144345237](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906144345237.png)

- #### [常见的缓存更新策略](https://xiaolincoding.com/redis/base/redis_interview.html#%E8%AF%B4%E8%AF%B4%E5%B8%B8%E8%A7%81%E7%9A%84%E7%BC%93%E5%AD%98%E6%9B%B4%E6%96%B0%E7%AD%96%E7%95%A5)

  1. 旁路缓存：（旁路缓存）策略是**最常用**的，应用程序直接与「数据库、缓存」交互，并负责对缓存的维护
  2. （读穿 / 写穿）策略
  3. （写回）策略：策略在更新数据的时候，只更新缓存，同时将缓存数据设置为脏的，然后立马返回，并不会更新数据库。对于数据库的更新，会通过批量异步更新的方式进行。

![image-20240906150236683](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906150236683.png)

![image-20240906150701110](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906150701110.png)

1. 先删缓存：（由于查询数据库和写入缓存的耗时较少，所以发生数据不一致的概率较高） 
   1. Thread1删除缓存
   2.  Thread2查询未命中之后，查询数据库，写入缓存
   3. Thread1 更新数据库

> [!NOTE]
>
> 解决方案： **延迟双删**
>
> ```
> #删除缓存
> redis.delKey(X)
> #更新数据库
> db.update(X)
> #睡眠
> Thread.sleep(N)
> #再删除缓存
> redis.delKey(X)
> ```

2. ✔️先操作数据库：（写入缓存耗时为微妙级别，发生概率较小）
   1. Thread1 查询，缓存未命中，查询数据库 
   2. Thread2 更新数据库，删除缓存
   3. Thread1 将旧数据写入缓存

- #### [如何保证数据库与缓存的一致性](https://xiaolincoding.com/redis/architecture/mysql_redis_consistency.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E4%B8%80%E8%87%B4%E6%80%A7)

#### 如何保证删缓存和更新数据库两个操作同时完成

- 方案一：**使用消息队列重试缓存删除**：消费者从消息队列中获取缓存删除的任务，尝试删除缓存。如果删除缓存失败，消息队列的重试机制会重新尝试，直到删除成功或达到最大重试次数。（消息可能重复消费，因此缓存删除操作需要幂等设计。）
- 方案二：**订阅 MySQL Binlog + 消息队列 + 重试缓存删除**：通过监听 MySQL 的 Binlog 日志，实时捕获数据库变更事件，触发缓存删除操作，并结合消息队列实现异步删除和失败重试。
- 方案三：**分布式事务（两阶段提交或 TCC 模型）**





### 7.2.2 缓存穿透/雪崩/击穿

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

## 7.3 秒杀业务

### 7.3.1 reids生成全局ID

常在分布式架构下使用，满足：唯一、高可用、高性能、递增(方便索引插入)、安全性(规律性不能太明显)

![image-20240906165830665](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240906165830665.png)

### 7.3.2 超卖问题

- 解决炒卖问题-> 加锁（悲观锁 or 乐观锁）

![image-20240920231851886](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240920231851886.png)

- 乐观锁中如何在更新数据时得知其他线程对数据有没有做修改？？？

  1. 版本号法

     > 以共同编辑文档为例
     >
     > - 由于发生冲突的概率比较低，所以先让用户编辑文档，但是浏览器在下载文档时会记录下服务端返回的文档版本号；
     > - 当用户提交修改时，发给服务端的请求会带上原始文档版本号，服务器收到后将它与当前版本号进行比较，如果版本号不一致则提交失败，如果版本号一致则修改成功，然后服务端版本号更新到最新的版本号。
  
  2. CAS法
  
     ![image-20240920232440440](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240920232440440.png)
  
     乐观锁失败率较高，每次compare时只需要判断库存是否大于0即可，不用判断是否发生改变

### 7.3.3 一人一单限制

查询订单记录里的该用户id数量是否>0

- 线程安全问题：由于是做查询不是做修改，所以只能用悲观锁方案，不能用乐观锁方案，但是只需要锁用户id

- 将整个查询id、扣减库存、添加订单加锁 



> [!WARNING]
>
> 集群模式下，锁只对当前进程有效  ---> **[分布式锁](##7.4 Redis分布式锁)**  



### 7.3.4 redis优化秒杀业务

- 将库存判断和一人一单判断放到redis里实现

![image-20240921154902002](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921154902002.png)



## 7.4 Redis分布式锁

### 7.4.1 基本实现

思路：

> [!NOTE]
>
> - 利用set nxex获取锁，并设置过期时间，保存线程标示
> - 释放锁时先判断线程标示是否与自己一致，一致则删除锁

 ![image-20240921141327908](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20240921141327908.png)

- 问题：

  - 超时时间小于业务所需时间，业务完成时有可能释放其他线程的锁（所以释放锁时需要判断key的value是不是自己的线程id）

    ![image-20240921142625162](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921142625162.png)

  - 多台主机的线程id可能重复，所以可以用uuid + 线程id做为value

  - 每当删除锁时，需要判断是否是自己创建的锁

  - > [!WARNING]
    >
    > 获取锁标识并判断是否一致  与  释放锁 需要是原子的
    >
    > ![image-20240921145719906](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921145719906.png)



###  7.4.2 Reids执行Lua脚本实现原子操作

> ==Redis的Lua脚本执行是原子的==，因为Redis在执行Lua脚本时将脚本视为一个不可分割的事务。Redis的内部实现通过以下方式保证了这一点：
>
> 1. **单线程执行**：Redis是单线程模型，在执行Lua脚本时，会阻塞其他命令的执行，直到脚本执行完成。
> 2. **脚本执行的不可中断性**：一旦Lua脚本开始执行，它会完整地执行，直到结束。执行期间，不会有其他命令打断它。
> 3. **事务性质**：所有在Lua脚本中发出的Redis命令（如`GET`、`SET`、`INCR`等）都是作为一个整体执行的，要么全部成功，要么全部失败

![image-20240921144206913](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921144206913.png)

![image-20240921145018728](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921145018728.png)

```go
func acquireLock(ctx context.Context, key string, value string, expiration time.Duration) (bool, error) {
	// 使用NX和PX选项来设置分布式锁
	result, err := rdb.SetNX(ctx, key, value, expiration).Result()
	if err != nil {
		return false, err
	}
	return result, nil
}

// 释放锁
func releaseLock(ctx context.Context, key string, value string) (bool, error) {
	// Lua脚本：仅当key的值与给定value相同时才删除
	script := redis.NewScript(`
	if redis.call("GET", KEYS[1]) == ARGV[1] then
		return redis.call("DEL", KEYS[1])
	else
		return 0
	end`)

	// 执行Lua脚本
	result, err := script.Run(ctx, rdb, []string{key}, value).Result()
	if err != nil {
		return false, err
	}

	// 判断结果是否为1（1表示删除成功，0表示未删除）
	return result == int64(1), nil
}
```





### 7.4.3 基于Redis的分布式锁优化 —— Redisson

> 基于setnx实现的分布式锁存在的问题

- 不可重入：同一个线程无法多次获取同一把锁
- 不可重试：获取失败，没有重试机制
- 超时释放：**锁续期机制**：在某些情况下，如果任务执行时间较长，可以通过实现锁的续期机制（例如定期重新设置锁的过期时间）来保持锁的有效性。
- 主从一致性：



## 7.5 Redis消息队列

- [rabbitMQ](../RabbitMQ.md)



### 7.5.1 list实现

LPUSH + BRPOP 实现

> - 单消费者
> - 无法保证可靠性

### 7.5.2 发布订阅实现

![image-20240921203124380](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921203124380.png)

> 不支持数据持久化
>
> 无法保证可靠性
>
> 消息堆积有上限，超出时数据丢失（消费者无法接收到离线时的数据）

### 7.5.3 [Streams](https://redis.io/docs/latest/commands/?group=stream)

1. `XADD`生产

![image-20240921204204306](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921204204306.png)

2. `XREAD`消费

通过`XREAD`的`$`读取，可能会出现==消息漏读==的问题，因为`$`只会读取最新消息

![image-20240921204701258](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921204701258.png)

3. `XGROUP`消费者组

![image-20240921205309401](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240921205309401.png)

- `XGROUP CREATE`
- `XREADGROUP`



## 7.6 达人探店

