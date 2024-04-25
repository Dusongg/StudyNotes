# ==[Redis参考手册](https://redis.io/docs/latest/commands/)==

# 1 基础认识

- NoSQL与SQL

  ![image-20240425234137088](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20240425234137088.png)

- 特征

![image-20240425234614363](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20240425234614363.png)

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

9. `flushall`：清除redis上的所有键值对

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

### hash



### list



### set



### zset

## 1.4 单线程模型

redis虽然是单线程模型，为什么效率却这么高

1. 访问内存，MySQL则多是访问硬盘
2. 核心功能简单
3. 单线程模型，避免了多线程带来的开销
4. 底层使用epoll





