https://gitlab.0voice.com/2310_vip/mysql_pool_impl

mysql网络模型：

3306端口

通过select监听listenfd

![image-20240321212523489](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321212523489.png)

- 维持tcp长连接，
  - keepalive  
  - 心跳维持链接：mysql_ping/mysql_pong ,通过发送心跳包还能确定线程是否活跃（mysql内部是否产生了死锁）

![image-20240321213301168](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321213301168.png)



# 同步/异步 链接



![image-20240321220805826](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321220805826.png)

异步：向mysql请求，之后回来指向313的语句，在某个时刻调用callback处理mysql返回回来的结果



![image-20240321222913214](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321222913214.png)

## 同步链接池

- 当前线程从连接池中获取可用链接（未被上锁的链接）
-  连接池里有多个链接，每个链接配一把锁，线程通过轮询的方式请求链接，并执行sql语句，阻塞等待返回结果

![image-20240321231000012](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321231000012.png)



## 异步连接池

- 

![image-20240321231259262](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321231259262.png)

# 裸SQL语句 / 预处理SQL语句

- 裸SQL语句 ： 每一个执行都要经历从 **<u>连接器 - 查询缓存 - 解析SQL - 预处理-优化器优化-指定执行计划 - 到存储引擎查询</u>**，几个步骤
- 预处理SQL语句：将前面的步骤预处理，之后每次执行相同的语句只需要执行最后在存储引擎查询的步骤



# SQL接口

![image-20240321233235441](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321233235441.png)

# 如何知道异步的返回值是哪个链接的结果？

![image-20240321234954967](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240321234954967.png)