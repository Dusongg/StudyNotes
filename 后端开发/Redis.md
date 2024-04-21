# 1 初始工作

- 安装：`sudo apt install redis`

- 配置文件

  ```
  bind 0.0.0.0 ::1    #将本地环回改为任意ip
  
  protected-mode no   #开启后其他主机才能访问
  ```

  配置完后重启服务器：`service redis-sever restart`

  查看运行状态：`service redis-server status`

  ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240421202357236.png)

- 链接

  ```bash
  redis-cli
  ```

- 退出：`quit` 或 `ctrl + d`

# 2 通用命令

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

6. `ttl`：(time to live)

## 2.1 全局命令

全局命令：能够搭配任意一个数据结构使用的命令

1. [`keys`](https://redis.io/docs/latest/commands/keys/)

```
KEYS pattern
```

- pattern

  ![image-20240421224000208](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240421224000208.png)







