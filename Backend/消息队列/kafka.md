# kafka

# 什么是kafka

重topic，将单个topic分为多个partition提高**并发性**，将多个partition分不在多个broker上提高**拓展性**，为了提高**可用性**为partition加了多个副本，<u>为了协调管理kafka集群数据消息添加组件zookeeper</u>（之后被kRaft替代）

# 什么是kraft

**KRaft**（Kafka Raft）是 Kafka 自身实现的一个新的 **元数据管理协议**，旨在替代旧有的 Zookeeper 作为 Kafka 集群的元数据存储和管理机制。

**KRaft 的工作原理：**

​	•	在 KRaft 模式下，Kafka 集群的元数据由一个特殊的元数据节点进行管理，这些节点使用 Raft 协议进行同步。

​	•	领导者节点（Leader）管理元数据的写入，其他跟随者节点（Followers）复制领导者的元数据。

​	•	如果领导者节点出现故障，Raft 协议会通过选举机制选出新的领导者，确保元数据的高可用性。



