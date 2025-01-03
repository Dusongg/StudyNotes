# 认识

- ES简介：Elasticsearch(后面简写ES)是一个分布式、高扩展、近实时的搜索与数据分析引擎，它能很方便的使大量数据具有搜索、分析和探索的能力，充分利用Elasticsearch的水平伸缩性，能使数据在生产环境变得更有价值。
- ES生态：
  - **Kibana**：Kibana是Elastic Stack产品中的一款<u>可视化工具</u>，支持柱状图、线状图、饼图、旭日图等多种图形，还可以使用Vega 语法来设计独属于我们自己的可视化图形。
  - **Beats**：Beats是一个<u>轻量型采集器的平台</u>，集合了多种轻量级的、单一的数据采集器，几乎可以兼容所有的数据类型，这些采集器可以从成千上万的系统中采集数据并向Logstash和Elasticsearch发送数据。
  - **LogStash**：Logstash是开源的<u>服务器端数据处理管道</u>，能够同时从多个来源采集数据，转换数据，然后将数据发送到您最喜欢的存储库中，一般就是发送到Elasticsearch当中。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223174301416.png" alt="image-20241223174301416" style="zoom:67%;" />

## 搜索术语

![image-20241223174340399](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223174340399.png)

## ES执行流程



<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223174429119.png" alt="image-20241223174429119" style="zoom:67%;" />





# ES组成

## Term Index

将term dictionary的key构造FST（前缀树），由于内存占用小，可以存放在内存中加速搜索

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223191104320.png" alt="image-20241223191104320" style="zoom:50%;" />

> <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223191859915.png" alt="image-20241223191859915" style="zoom:50%;" />

## Stored Field

Stored Field用于存放完整的文档内容

### Doc Values

**Doc Values** 是一种为字段预先构建的、面向列式存储的数据结构，主要用于支持**快速排序**、**聚合**和**脚本计算**等操作。

- **列式存储**（如 Doc Values）将同一字段的所有文档值存储在一起。对于需要操作单一字段（例如排序、聚合）的场景，列式存储更加高效。

## Segment

在 Elasticsearch 中，**Segment（段）** 是 <u>Lucene 的一种基础存储单元，也是 Elasticsearch 索引数据的核心组成部分</u>。每个索引的文档都会被分成多个 Segment，这些 Segment 是不可变的，并存储着倒排索引和其他数据结构，用于快速查询。

- segment是具备查询功能的最小单元

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20241223194826034.png" alt="image-20241223194826034" style="zoom:50%;" />

1. **segment只读**

多个文档生成一个segment，如果有新的文档添加进来，由于涉及修改segment内部多个结构，影响效率，所以新添加的文档会生成一个新的segment，对于多个segment只需并发读就行

2. **segment merge（段合并）**

为了优化查询性能，减少 Segment 数量，Elasticsearch 会定期触发 **Segment 合并**。合并时，多个小 Segment 会被合并为一个更大的 Segment，同时清理被删除文档的空间。



## Segment与索引的关系

> [!NOTE]
>
> **索引** 是 Elasticsearch 中的逻辑命名空间，是数据的存储、组织和管理单元。一个索引可以包含许多文档，每个文档以 JSON 格式存储。
>
> - 索引在底层由多个 **分片（Shard）** 组成，每个分片实际上是一个独立的 **Lucene 索引**。
> - ES中的索引相当于关系型数据库中的表

- **索引、分片与 Segment 的层次关系**

  ```
  Index (逻辑命名空间)
  └── Shards (分片，分布式存储)
      └── Segments (物理存储单元)
  ```

  - **副本(Replica)**

1. **什么是副本**：**副本** 是主分片的完整拷贝，用于提高数据的==可用性==和==查询性能==。每个主分片可以有一个或多个副本分片，分布在不同的节点上。

2. **副本的作用**：
   1. **高可用性（High Availability**：如果主分片所在的节点发生故障，副本分片可以接管，确保数据不会丢失。Elasticsearch 会自动管理主分片和副本分片的切换。
   2. **提高查询性能（Query Throughput**：副本分片可以分担查询请求，从而提高并发查询的性能。

3. 配置：

   ```json
   PUT /my_index
   {
     "settings": {
       "number_of_shards": 3,        // 主分片数量，确定索引被分成几部分
       "number_of_replicas": 1       // 每个主分片的副本数量(默认是1)
     }
   }
   //总分片数 = 主分片数 × (1 + 副本数量)
   ```



# ES常见面试题

​	1.	**Elasticsearch 的工作原理是什么？为什么需要分片和副本？**

​	2.	**如何设计一个搜索功能？涉及到分词器、索引优化等知识点。**

​	3.	match **和** term **查询的区别是什么？**

​	4.	**如何实现一个电商商品的模糊搜索？如何优化查询性能？**

​	5.	**怎样设计一个高可用的 Elasticsearch 集群？如何处理节点故障？**

​	6.	**如何实现实时日志分析和监控？**

​	7.	**如果数据量非常大（亿级），如何设计分片？如何避免热点分片？**