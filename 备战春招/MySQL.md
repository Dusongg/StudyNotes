 

# 基础

## MySQL架构

- 数据页：将整个表分为多个16kb的数据页，加速读取效率
- 索引
- `Buffer Pool`：在磁盘与应用之间加一层  **进程内缓存**，查询时优先缓存（操作系统有文件缓存，为什么还要有Buffer Pool）
- 自适应哈希索引：记录每个数据页的查询频率，建立哈希表，<u>key 为查询值，value为数据页地址</u>
- `Change Buffer`：（包含在Buffer Pool中）**Change Buffer（更改缓冲区）** 是InnoDB存储引擎用于优化磁盘I/O性能的一种机制。它的作用是临时存储对二级索引页（非聚簇索引）的修改操作，延迟将这些修改写入磁盘，从而减少磁盘I/O操作并提高写入性能。
  - 写操作：当非聚簇索引执行插入/更新/删除操作时，检测该索引页是否加载到了内存缓冲中，如果存在则更新，不过不存在则将修改记录到Change Buffer中
  - 合并操作：合并操作会将 Change Buffer 中的修改与对应的索引页合并并写入磁盘。（查询索引页/后台定期执行/数据库关闭）
-  `Undo Log`：记录旧数据，定期写入到磁盘的undo log文件中，以便在事务回滚时能够恢复数据的原始状态
- `Redo Log`: 将事务中更新数据的操作都写入到Redo log Buffer中，当事务提交时刷新到Redo log文件中，防止数据库进程崩溃导致buffer pool的数据没有刷新到磁盘中（由于redo log是**顺序写**磁盘，比buffer pool中的**随机写**性能高很多 —— WAL（write ahead logging））
- 存储引擎：对外提供API接口，对内管理上述数据存储操作
- server层：链接管理、分析器、优化器、执行器(预处理 + 优化+ 执行)
- `Bin Log`：server层会讲所有历史的变更操作记录到binlog中（**与redo log的区别？**）

![image-20250124下午82249028](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8882249028.png)

## 数据库查询更新流程

![image-20250124下午83624930](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%8883624930.png)

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%88113105092.png" alt="image-20250126下午113105092" style="zoom:50%;" />

## 一条SQL语句执行顺序







## 行溢出怎么处理

当发生行溢出时，在记录的真实数据处只会保存该列的一部分数据，而把剩余的数据放在「溢出页」中，然后**真实数据处用 20 字节存储指向溢出页的地址**，从而可以找到剩余数据所在的页。大致如下图所示。

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/row_format/%E8%A1%8C%E6%BA%A2%E5%87%BA.png" alt="img" style="zoom:50%;" />

## 数据页结构

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8A%E5%8D%88123759566.png" alt="image-20250126上午123759566" style="zoom:50%;" />

InnoDB 里的 B+ 树中的**每个节点都是一个数据页**，结构示意图如下：

![image-20250126上午123814790](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8A%E5%8D%88123814790.png)

> **B+树叶子节点的数据结构**
>
> **(1) 页（Page）的基本结构**
>
> ​	•	**InnoDB 页的大小**：默认 16 KB。
>
> ​	•	每个页（Page）包含多条记录（Record）。
>
> ​	•	页头和记录的关键部分：
>
> ​	•	页头（Page Header）：管理页的元信息。
>
> ​	 	 页目录：
>
> ​	•	记录（Record）：存储索引键值和数据。
>
> 
>
> **(2) 记录（Record）的结构**
>
> 每条记录包含以下重要部分：
>
> ​	1.	**主键值**（对于聚簇索引）或索引列值（对于二级索引）。
>
> ​	2.	**下一条记录的指针**（链表结构，用于记录之间的顺序）。
>
> ​	3.	**隐藏列**（例如事务 ID、回滚指针等，用于并发控制和事务管理）。
>
> ​	4.	**数据列**（在聚簇索引中存储行数据，在二级索引中存储主键值）。

# 索引

## B+ vs B树

1. **磁盘读写效率**

   **B+ 树**的非叶子节点不存储实际数据，<u>单个节点可以容纳更多的索引信息</u>（指针数量增加）。这减少了树的高度，从而减少磁盘 I/O。

   对于 B 树，节点中存储了数据和索引，导致每个节点的大小较大，树的高度增加，磁盘访问次数更多。

2. **插入和删除**

   B+树只需要对叶子节点，B树可能操作多层节点

3. **范围查询**







## 索引优化

1. 使用前缀索引

```mysql
CREATE INDEX idx_name ON users (name(10));
```

2. 覆盖索引
3. 避免索引失效
4. 使用**分区表**：对于超大表，可以<u>结合分区表和索引使用，分散数据以减少扫描范围</u>。

```mysql
CREATE TABLE orders (
    id INT NOT NULL,
    order_date DATE NOT NULL
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p0 VALUES LESS THAN (2000),
    PARTITION p1 VALUES LESS THAN (2010),
    PARTITION p2 VALUES LESS THAN (2020)
);
```

5. 优化复合索引(联合索引)顺序
6. 将选择性高的列放在前面（选择性 = 不重复值的数量 / 总记录数）。

> 假设数据特点如下：
>
> 1. city 列有 5 个不同值（如北京、上海等），选择性为 0.0005。
>
> 2. age 列有 50 个不同值（如 20 到 70 岁），选择性为 0.005。
>
> 
>
> 在这种情况下，应将 age 放在复合索引的第一列，city 放在后面：
>
> ```mysql
> CREATE INDEX idx_age_city ON users (age, city);
> ```

7. 根据查询条件的使用频率调整顺序。例如，WHERE column1 = ? AND column2 > ? 时，将 column1 放在前面。
8. 索引最好设置为NOT NULL
9. 主键索引最好是自增的：防止插入数据导致页分裂进而导致内存碎片



## 什么字段适合用索引

- 选择性高的字段（重复数量较小）
- 常用于where语句的字段
- 有唯一性约束的字段
- 常用于group by / distinct / join / order by

### 什么字段不适合

1. **选择性低的字段**：如 gender、status 等，仅有少量不同值的字段。
2. **频繁更新的字段**：如计数器字段，索引维护开销大。
3. **很长的字段**：如 TEXT 或 BLOB 类型，索引效率低。
4. **小表字段**：表的数据量很小时，扫描全表比使用索引更快。



## 索引失效

- **左模糊匹配**
- **对索引使用函数**（Mysql8.0之后支持函数索引：对某个字段的函数值建立索引） or 对索引进行表达式运算
- **对索引进行隐式转换**
- **联合索引非最左匹配**
- **在where子句中使用or**





## 一行数据如何存储

![image-20250124下午114908064](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%88114908064.png)



# 事务

- 事务的特性（ACID）

  - 原子性：一个事务中的所有操作，要么全部完成，要么全部不完成 —— undo log（回滚日志）

  - 一致性：是指事务操作前和操作后，数据满足完整性约束，数据库保持一致性状态

  - 隔离性：多个事务间操作隔离 —— MVCC / 锁

  - 持久性：事务处理之后，对数据的操作是永久的 —— redo log（重做日志）

- 事务引发的问题
  - 脏读：读到未提交的数据（数据会滚导致前后不一致）
  - 不可重复读：两次读的**数据值**不一致
  - 幻读：两次读的**数据量**不一致

- 隔离级别
  - 读未提交 —— 造成脏读
  - 读提交 —— 每个语句执行前生成快照 —— 造成不可重复度
  - 可重复读 —— 启动事务时生成快照
  - 串行化

## 不同命令开启事务的时机不同

- `begin/start transaction` ：在执行这个命令后，执行了第一条 select 语句，才是事务真正启动的时机；
- `start transaction with consistent snapshot` :马上启动事务

## Read View工作原理

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250125%E4%B8%8B%E5%8D%88114936838.png" alt="image-20250125下午114936838" style="zoom:50%;" />

- trx_id < min_trx_id：可见

- trx_id >= max_trx_id：不可见

- trx_id在min_trx_id与max_trx_id之间：判断是否在m_ids列表中，如果在则不可见，反之



## 发生幻读的场景

- 快照读怎么解决幻读？
- 当前读怎么解决幻读？

> MySQL 里除了普通查询是快照读，其他都是**当前读**，比如 update、insert、delete，这些语句执行前都会查询最新版本的数据，然后再做进一步的操作。



### case 1 ： 更新别人插入的数据

事务A快照读一个没有的数据

事务B插入一条数据

事务A更新事务B插入的数据

事务A快照读第一步这个数据

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8A%E5%8D%88120031608.png" alt="image-20250126上午120031608" style="zoom:50%;" />

### case 2 ： 快照读—〉insert—〉当前读

- T1 时刻：事务 A 先执行「快照读语句」：select * from t_test where id > 100 得到了 3 条记录。
- T2 时刻：事务 B 往插入一个 id= 200 的记录并提交；
- T3 时刻：事务 A 再执行「当前读语句」 select * from t_test where id > 100 for update 就会得到 4 条记录，此时也发生了幻读现象。





# 锁

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8A%E5%8D%88120547954.png" alt="image-20250126上午120547954" style="zoom: 33%;" />

**加锁的对象是索引，加锁的基本单位是 next-key lock**，它是由记录锁和间隙锁组合而成的，**next-key lock 是前开后闭区间，而间隙锁是前开后开区间**。

- 加锁加在两索引的右边那一个，正无穷加在特殊标识`supremum pseudo-record` 上

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8A%E5%8D%88122556330.png" alt="image-20250126上午122556330" style="zoom:50%;" />

## 如何判断锁冲突

- 加锁位置：索引 ——〉B+树叶子节点 ——〉页 + 目标记录

> **判断是否加锁的过程**
>
> 当事务试图访问某条记录时，InnoDB 会通过以下步骤判断是否已被加锁：
>
> **(1) 定位目标记录**
>
> ​	•	事务通过查询条件定位到记录所在的 **B+树叶子节点**。
>
> ​	•	根据记录的主键或索引键值找到目标记录。
>
> 
>
> **(2) 查找锁哈希表**
>
> ​	•	InnoDB 使用目标记录所在的**页号和记录位置**在锁哈希表中查找是否已有锁。
>
> ​	•	关键数据结构：
>
> ​	•	页号（page_no）：记录所在的 B+树页编号。
>
> ​	•	记录位置（heap_no）：记录在该页中的位置。
>
> ​	•	如果==锁哈希表==中存在匹配项，则表明该记录已被加锁。
>
> 
>
> **(3) 检查锁冲突**
>
> ​	•	如果找到锁，检查锁的类型（共享锁或排他锁）和当前事务是否兼容：
>
> ​	•	**共享锁（S 锁）**：允许多个事务同时读取。
>
> ​	•	**排他锁（X 锁）**：禁止其他事务的任何访问。
>
> ​	•	如果锁冲突，事务将进入等待队列或触发死锁检测。

- 加锁成功：将锁对象插入到事务的==锁队列==中。



## 死锁

![image-20250126下午30640397](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%8830640397.png)

1. 相同间隙锁可以共存：**间隙锁的意义只在于阻止区间被插入**，因此是可以共存的。**一个事务获取的间隙锁不会阻止另一个事务获取同一个间隙范围的间隙锁**（相同next-lock会被阻塞）

2. 同一个事务下不会被锁阻塞





# 日志

## undo log

- undo log刷盘：

> undo log 和数据页的刷盘策略是一样的，都需要通过 redo log 保证持久化。
>
> buffer pool 中有 undo 页，对 undo 页的修改也都会记录到 redo log。redo log 会每秒刷盘，提交事务时也会刷盘，数据页和 undo 页都是靠这个机制保证持久化的。



## redo log

- **redo log** 是一种用于实现 **事务持久性** 和 **故障恢复** 的日志机制，存放数据页做了什么修改

- WAL（Write-Ahead Logging）：**MySQL 的写操作并不是立刻写到磁盘上，而是先写日志，然后在合适的时间再写到磁盘上**。

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%8893719333.png" alt="image-20250126下午93719333" style="zoom:50%;" />

### redo log 的作用

1. **持久性保证**：即使在数据库崩溃后，redo log 也能通过重新应用日志中的操作来恢复数据，使数据库处于一致的状态。

2. **性能优化**：事务提交时，不需要立即将数据刷入磁盘，而是先记录到 redo log 中，这种方式称为**预写日志（Write-Ahead Logging, WAL）**。

### redo log 的工作原理

1. **写入 redo log buffer**：当事务执行时，数据的更改操作会先写入到内存中的 **redo log buffer**。
2. **刷入 redo log 文件**：数据最终会以顺序写的方式刷入磁盘上的 redo log 文件（称为 `ib_logfile0` 和 `ib_logfile1`）。
3. **事务提交**：在事务提交前，InnoDB 会确保与该事务相关的所有 redo log 都已经写入到磁盘，保证数据的安全性。
4. **崩溃恢复**：如果数据库异常宕机，InnoDB 会在重启时从 redo log 中读取记录，重新应用尚未完成的操作。

- redo log与 undo log的区别

  - redo log 记录了此次事务「**修改后**」的数据状态，记录的是更新**之后**的值，**主要用于事务崩溃恢复，保证事务的持久性**。

  - undo log 记录了此次事务「**修改前**」的数据状态，记录的是更新**之前**的值，**主要用于事务回滚，保证事务的原子性**。

### redo log刷盘时机

- MySQL 正常关闭时；
- 当 redo log buffer 中记录的写入量大于 redo log buffer 内存空间的一半时，会触发落盘；
- InnoDB 的后台线程每隔 1 秒，将 redo log buffer 持久化到磁盘。
- 每次事务提交时都将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘（这个策略可由 `innodb_flush_log_at_trx_commit` 参数控制，下面会说）。

>  `innodb_flush_log_at_trx_commit` :
>
> - 当设置该**参数为 0 时**，表示每次事务提交时 ，还是**将 redo log 留在 redo log buffer 中** ，该模式下在事务提交时不会主动触发写入磁盘的操作。
> - （默认）当设置该**参数为 1 时**，表示每次事务提交时，都**将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘**，这样可以保证 MySQL 异常重启之后数据不会丢失。
> - 当设置该**参数为 2 时**，表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log **写到 redo log 文件，注意写入到「 redo log 文件」并不意味着写入到了磁盘**，因为操作系统的文件系统中有个 Page Cache





### redo log 文件满了怎么办

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%88101949127.png" alt="image-20250126下午101949127" style="zoom:50%;" />



## bin log

### binlog与redo log区别

- 格式
- 适用对象：binlog是server层概念，redo log是存储引擎层概念
- 写入方式
- 用途
