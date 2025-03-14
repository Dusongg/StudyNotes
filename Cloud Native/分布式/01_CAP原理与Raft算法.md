![image-20250225下午23527663](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250225%E4%B8%8B%E5%8D%8823527663.png)

![image-20250225下午23838954](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250225%E4%B8%8B%E5%8D%8823838954.png)

# CAP理论

- **C Consistency 一致性**：所有节点在同一时间看到的数据都是一致的

  - 即时一致性问题：要求服务端做到写入立即可读 ![image-20250110下午124656629](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%88124656629.png)

  - 顺序一致性问题：

    <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250108220544130.png" alt="image-20250108220544130" style="zoom: 80%;" />

- **A: Availability 可用性**：可用性意味着每个请求都会得到一个有效的响应，无论请求是否成功。即使某些节点发生故障，系统仍然可以继续对外提供服务。

- **P: Parition tolerance 分区容错性**：即使某些节点失去联系，系统也不会完全崩溃。





# Raft算法

## 前置知识

### 相关术语

- 最终一致性：保证数据在一定的时间内会达到一致性，但在此期间，系统可能会存在数据不一致的状态。
- 即时一致性：即时一致性是一种严格的一致性模型，它要求系统中的所有副本在任何时刻都保持一致。

![image-20250110下午10533737](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%8810533737.png)

### 一主多从、读写分离



### 预写日志和状态机

#### 预写日志

预写日志是 Raft 算法中的日志机制，用于记录状态更新之前的所有操作。这些日志条目在被应用到状态机之前会先写入到持久化存储中，确保即使系统崩溃，也可以通过重放日志恢复状态。

- **操作序列记录**：记录了所有的客户端请求（如写入操作），这些请求会<u>按顺序</u>被提交到状态机。
- **一致性保证**：所有节点上的日志副本保持一致，只有当日志条目被大多数节点确认后才会提交。
- 每条日志条目包含以下信息：
  - **索引号**（Log Index）：标识日志条目的位置。
  - **任期号**（Term Number）：表示该条目属于哪个任期。
  - **操作**（Command）：存储需要执行的状态变更命令。
  - 在日志条目被复制到多数节点并且被提交（commit）后，才可以应用（apply）到状态机

#### 状态机

状态机是 Raft 算法中执行具体操作的组件，它从提交的日志条目中获取命令并执行，更新系统的实际状态。

### 两阶段提交

![image-20250109011718720](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250109011718720.png)

1. 第一阶段：提议（proposal），



2. 第二阶段：提交（commit）



### 领导者选举

1. fllower如何感知leader挂了

​	心跳包

2. 什么样的fllower有资格成为leader

   数据足够新，通过广播拉票一半以上

### 任期与日志索引 

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%8833415555.png" alt="image-20250110下午33415555" style="zoom:50%;" />

通过任期与日志索引**{term, index}**可以唯一的标识一个操作



## Raft节点及交互

### 广播心跳

- leader每隔一段时间发送心跳，带上自身的term、最新的commitIndex，同时接收到的follower会更新自己的CommitIndex，并在合适的时机将已提交的预写日志应用到状态机中

  ![image-20250110下午93144035](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%8893144035.png)

### 竞选拉票







## 集群新增节点

- 新增节点在配置变更期间，

  - <u>不能参与leader选举</u>（否则可能会出现**脑裂**的请况），

  ![image-20250110下午110421588](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%88110421588.png)

  - <u>不能参与提议阶段</u>

![image-20250110下午110830892](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250110%E4%B8%8B%E5%8D%88110830892.png)





## Q&A

### 提议达成多数派认可就能提交？

一个极端的case：

> 背景：集群中存在 s1-s5 5 个节点，只需要 3 个节点达成共识，即可形成多数派.
>
> （1）moment1：此时 leader 为 s1，term 为 1，s1 接受了一笔写请求，刚将其同步到 s1、s2，还未形成多数派时，s1 就宕机了；
>
> （2）moment2：s5 收获了 s3、s4、s5 的选票，当选 leader，接受了一笔写请求，只在本机完成预写日志的落盘就宕机了；
>
> （3）moment3：s1 收获了 s1、s2、s3、s4 的选票，重新当选 leader，继续推进 term = 1 时那笔遗留写请求的提交，成功将其同步到了 s1、s2、s3，获得多数派的认同，于是提交这笔写请求. 提交之后，s1 又宕机了.
>
> （4）moment4：s2 由于遗留了一笔 term 为 2 的日志 term，领先集群所有节点，因此可以收获集群所有节点的选票. 于是 s5 再度当选 leader，继续推进 term 为 2 时遗留的写请求，由于这笔日志的 index 与第（3）步中 s1 同步日志的 index 相同，又因为其 term 值更大，最终会覆盖 s1、s2、s3 中的老日志，**这就导致一笔已经被 s1 提交的日志最终被 s5 回滚了.**

因此，raft新增如下限制：

- 新上任的leader需要至少完成一笔本任期内的写请求才能够执行提交操作

### 如何解决网络分区引发的无意义选举

> 每个 candidate 发起真实选举之前，会有一个提前试探的过程，试探机制是向集群所有节点发送请求，只有得到多数派的响应，证明自己不存在网络环境问题时，才会将竞选任期自增，并且发起真实的选举流程.

