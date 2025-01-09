# CAP理论

- **C Consistency 一致性**：所有节点在同一时间看到的数据都是一致的

  - 即时一致性问题：要求服务端做到写入立即可读 

    <img src="C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250108220457140.png" alt="image-20250108220457140" style="zoom:50%;" />

  - 顺序一致性问题：

    <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250108220544130.png" alt="image-20250108220544130" style="zoom: 80%;" />

- **A: Availability 可用性**：可用性意味着每个请求都会得到一个有效的响应，无论请求是否成功。即使某些节点发生故障，系统仍然可以继续对外提供服务。

- **P: Parition tolerance 分区容错性**：即使某些节点失去联系，系统也不会完全崩溃。





# Raft算法

## 相关术语

- 最终一致性：保证数据在一定的时间内会达到一致性，但在此期间，系统可能会存在数据不一致的状态。
- 即时一致性：即时一致性是一种严格的一致性模型，它要求系统中的所有副本在任何时刻都保持一致。



## 一主多从、读写分离



## 预写日志和状态机



## 两阶段提交

![image-20250109011718720](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250109011718720.png)

1. 第一阶段：提议（proposal），



2. 第二阶段：提交（commit）

