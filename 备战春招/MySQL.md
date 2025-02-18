 

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

```mysql
SELECT col1, SUM(col2) AS total
FROM table1
WHERE col3 = 'value'
GROUP BY col1
HAVING total > 100
ORDER BY total DESC
LIMIT 10;
```

**执行顺序解释**

​	1.	**FROM table1**

加载 table1 的所有数据。

​	2.	**WHERE col3 = ‘value’**

筛选出 col3 等于 'value' 的行。

​	3.	**GROUP BY col1**

按照 col1 分组，将同一组的行归为一组。

​	4.	**SELECT col1, SUM(col2) AS total**

计算每组中 col2 的总和，返回 col1 和聚合结果 total。

​	5.	**HAVING total > 100**

对分组后的结果进行过滤，保留 SUM(col2) 大于 100 的组。

​	6.	**ORDER BY total DESC**

按照聚合结果 total 的降序排列。

​	7.	**LIMIT 10**

返回前 10 条结果。





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



## 查询数据如何优化

**1️⃣ 索引优化**

- 选择合适索引
- 避免全表扫描，避免索引失效
- 覆盖索引

**2️⃣SQL语句优化**

- **不要用 SELECT \***，减少不必要的 I/O。
- **避免 ORDER BY 和 GROUP BY 临时建表**：适当使用**索引**，让 ORDER BY 或 GROUP BY 直接使用索引排序，而不是临时表排序。
- **避免 HAVING 过滤数据，尽量使用 WHERE**，减少不必要的分组计算。

**3️⃣ 事务 & 锁优化**

- 减少锁的范围
- 避免长事务

**4️⃣ 使用缓存**

- 使用 **Redis/Memcached** 缓存热点数据

**5️⃣ 数据库结构优化**

- **分区（Partitioning）**：大数据量表可以使用 **分区表**，加速查询。
- **分库分表**：当单表数据量过大（超过 1000 万行），考虑**水平分表（Sharding）** 或 **垂直拆分（Vertical Partitioning）**。

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

5. 优化复合索引(联合索引)顺序，将选择性高的列放在前面（选择性 = 不重复值的数量 / 总记录数）。

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
> 
> ```
>
> 例如：
>
>          [ (5, 2) (10, 1) (15, 3) ]
>               /       |       \
>        ---------  ---------  ---------
>        (1, 4)    (10, 1)   (20, 5)
>        (5, 2)    (10, 10)  (15, 3)
>        (5, 8)    (10, 20)  (20, 30)
>     对于 SELECT * FROM table WHERE A = 10 AND B = 20;：
>     1️⃣ B+ 树根据 A=10 快速定位到索引范围的起始位置。
>     2️⃣ 在叶子节点中，按 B 递增排序查找 B=20，通常只需少量顺序扫描。
>     3️⃣ 如果索引是覆盖索引，直接返回结果；否则，回表查询完整数据。
>     

7. 根据查询条件的使用频率调整顺序。例如，WHERE column1 = ? AND column2 > ? 时，将 column1 放在前面。
8. 索引最好设置为NOT NULL
9. 主键索引最好是自增的：防止插入数据导致页分裂进而导致内存碎片



## 什么字段适合用索引

- 选择性高的字段（重复数量较小）——有唯一性约束的字段
- 常用于where语句的字段
- 常用于group by / distinct / join / order by

### 什么字段不适合

1. **选择性低的字段**：如 gender、status 等，仅有少量不同值的字段。
2. **频繁更新的字段**：如计数器字段，索引维护开销大。
3. **很长的字段**：如 TEXT 或 BLOB 类型，索引效率低。
4. **小表字段**：表的数据量很小时，扫描全表比使用索引更快。



## 索引失效

- **左模糊匹配**

- **联合索引非最左匹配**

- **对索引使用函数**（Mysql8.0之后支持函数索引：对某个字段的函数值建立索引） or 对索引进行表达式运算

- **对索引进行隐式转换**

- **在where子句中使用or**——建议用 UNION ALL 替代。

  ```mysql
  SELECT * FROM users WHERE email = 'user@example.com' OR phone = '1234567890';
  #改为：
  SELECT * FROM users WHERE email = 'user@example.com'
  UNION ALL
  SELECT * FROM users WHERE phone = '1234567890';
  ```

  





## 一行数据如何存储

![image-20250124下午114908064](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250124%E4%B8%8B%E5%8D%88114908064.png)



## 什么是索引下推

**索引下推（Index Condition Pushdown, ICP）**：让存储引擎在**索引扫描阶段**先执行 WHERE 条件过滤，减少回表次数，提高查询速度。

**🔹 例子**

- **没有索引下推（ICP 之前的情况）**

假设有一个 users 表：

```mysql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    age INT,
    city VARCHAR(50),
    INDEX idx_name_age (name, age)
);

#查询
SELECT * FROM users WHERE name LIKE 'A%' AND age > 25;
```

**执行过程（无 ICP）：**

1️⃣ MySQL 先通过索引 idx_name_age 找到 **name LIKE ‘A%’** 的所有记录（索引可用）。

2️⃣ **存储引擎返回主键 ID**，然后**回表**取出完整的行数据。

3️⃣ MySQL 服务器再执行 age > 25 过滤。

❌ **问题：即使 age 不符合条件，也会先回表，占用不必要的 I/O。**



- **有索引下推（启用 ICP）**

**MySQL 5.6 及以上**，如果 WHERE 条件中的字段**全部或部分**在索引 idx_name_age 中，**存储引擎可以在索引层先执行过滤**，减少回表：

```
EXPLAIN SELECT * FROM users WHERE name LIKE 'A%' AND age > 25;
```

如果 Extra 字段显示：

```
Using index condition
```

✅ **优化点：**

1️⃣ MySQL 先通过索引 idx_name_age 找到 name LIKE 'A%' 的记录。

2️⃣ **存储引擎直接在索引中过滤 age > 25，减少不必要的回表**。

3️⃣ 只有符合 name LIKE 'A%' AND age > 25 的记录才会回表获取完整行数据。

🔥 **减少回表次数，提升查询效率！**





## explain如何实现





## MySQL里联合索引abc，a=a1 and b > b1 and c in (c1，c2)用到联合索引了吗？

### 优化方法：覆盖索引或索引下推（ICP）可提高性能。



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

m_ids：当前数据库中「活跃事务」的**事务 id 列表**

min_trx_id：当前数据库中「活跃事务」中事务 **id 最小的事务**

max_trx_id：这个并不是 m_ids 的最大值，而是**创建 Read View 时当前数据库中应该给下一个事务的 id 值**，也就是全局事务中最大的事务 id 值 + 1；



- trx_id < min_trx_id：可见

- trx_id >= max_trx_id：不可见

- trx_id在min_trx_id与max_trx_id之间：判断是否在m_ids列表中，如果在则不可见，反之



## 不可重复读和幻读的区别

**1️⃣ 幻读（Phantom Read）**

​	•	**概念**：指同一个事务中，两次范围查询的**结果集行数**发生变化。

​	•	例如：第一次查询 SELECT * FROM table WHERE age > 30 返回 5 行数据，之后另一个事务插入了一条符合条件的数据，再次查询时结果变为 6 行。

​	•	**触发原因**：其他事务对表进行了**插入或删除操作**，导致查询的结果集变化。

​	•	**发生场景**：多次范围查询（SELECT 带范围条件）。

**2️⃣ 不可重复读（Non-Repeatable Read）**

​	•	**概念**：指同一个事务中，多次读取同一行记录，数据内容发生了变化。

​	•	例如：第一次读取 id=1 的记录值是 100，另一个事务修改了 id=1 的值为 200，当前事务再次读取时发现数据变了。

​	•	**触发原因**：其他事务对某条记录**更新操作**导致数据变化。

​	•	**发生场景**：对**单行记录**多次读取。

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

## 全局锁



## 表级锁

1️⃣ **表锁**

```mysql
//表级别的共享锁，也就是读锁；
//允许当前会话读取被锁定的表，但阻止其他会话对这些表进行写操作。
lock tables t_student read;

//表级别的独占锁，也就是写锁；
//允许当前会话对表进行读写操作，但阻止其他会话对这些表进行任何操作（读或写）。
lock tables t_stuent write;
```

2️⃣ **元数据锁**

当我们对数据库表进行操作时，会自动给这个表加上 MDL：**事务执行期间，MDL 是一直持有的**

- 对一张表进行 CRUD 操作时，加的是 **MDL 读锁**；
- 对一张表做结构变更操作的时候，加的是 **MDL 写锁**；

⚠️申请 MDL 锁的操作会形成一个队列，队列中**写锁获取优先级高于读锁**，一旦出现 MDL 写锁等待，会阻塞后续该表的所有 CRUD 操作。



3️⃣ **意向锁**

```mysql
//先在表上加上意向共享锁，然后对读取的记录加共享锁
select ... lock in share mode;

//先表上加上意向独占锁，然后对读取的记录加独占锁
select ... for update;
```

意向共享锁和意向独占锁是表级锁，只会和共享表锁（lock tables ... read）和独占表锁（lock tables ... write）发生冲突。

**意向锁的目的是为了快速判断表里是否有记录被加锁**（不用去遍历表里的所有记录）

## 行级锁

**加锁的对象是索引，加锁的基本单位是 next-key lock**，它是由记录锁和间隙锁组合而成的，**next-key lock 是==前开后闭==区间，而间隙锁是前开后开区间**。

### 可重复读和读提交加锁方式

- **可重复读（REPEATABLE READ）**：在 **REPEATABLE READ** 隔离级别中，MySQL 默认会使用 next-key lock 来保证事务的隔离性，避免幻读问题。

- **读提交（READ COMMITTED）**：在 READ COMMITTED 中，MySQL **不会对间隙加锁**，只对命中的记录加记录锁（Record Lock）。







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

### 死锁产生的场景



**1️⃣间隙锁冲突**

![image-20250126下午30640397](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%8830640397.png)

1. 相同间隙锁可以共存：**间隙锁的意义只在于阻止区间被插入**，因此是可以共存的。**一个事务获取的间隙锁不会阻止另一个事务获取同一个间隙范围的间隙锁**（相同next-lock会被阻塞）

2. 同一个事务下不会被锁阻塞



**2️⃣ 不同顺序的更新操作**

```mysql
-- 事务A
UPDATE table1 SET ... WHERE id = 1;  -- 持有 table1.id=1 的锁
UPDATE table2 SET ... WHERE id = 2;  -- 尝试获取 table2.id=2 的锁

-- 事务B
UPDATE table2 SET ... WHERE id = 2;  -- 持有 table2.id=2 的锁
UPDATE table1 SET ... WHERE id = 1;  -- 尝试获取 table1.id=1 的锁
```



**3️⃣ 混合读写**

```mysql
-- 事务A
SELECT * FROM table1 WHERE id = 1 FOR UPDATE;  -- 锁住 table1.id=1
UPDATE table2 SET ... WHERE id = 2;

-- 事务B
SELECT * FROM table2 WHERE id = 2 FOR UPDATE;  -- 锁住 table2.id=2
UPDATE table1 SET ... WHERE id = 1;
```



4️⃣**唯一键冲突（Duplicate Key）**





### 如何避免MySQL死锁

1️⃣添加事务超时时间**设置合理的 innodb_lock_wait_timeout**

​	**MySQL 默认事务等待锁的时间是 50 秒 (innodb_lock_wait_timeout = 50)**，如果锁被长时间占用，可能导致大量事务阻塞。**建议将 innodb_lock_wait_timeout 设为一个合适的值**，避免长时间等待。

2️⃣ **保证事务访问表的顺序一致**

```mysql
-- 事务 A
BEGIN;
UPDATE table1 SET col = 'X' WHERE id = 1;
UPDATE table2 SET col = 'Y' WHERE id = 2;
COMMIT;

-- 事务 B
BEGIN;
UPDATE table2 SET col = 'Z' WHERE id = 2;
UPDATE table1 SET col = 'W' WHERE id = 1;
COMMIT;
```

3️⃣ 减少锁定行数

4️⃣ 合理使用索引，避免全表扫描

5️⃣ **降低隔离级别**：如使用 `READ COMMITTED` 减少间隙锁。





## 为什么update没加索引会锁全表

全表扫描导致锁住整个表，主要是因为在没有索引的情况下，MySQL 的 InnoDB 存储引擎需要扫描表中的所有行，逐一判断是否符合 WHERE 条件。在此过程中，为了保证事务的隔离性和数据一致性，InnoDB 会对扫描到的所有行加锁，这最终导致锁住整个表。

例如：

```mysql
UPDATE t SET col2 = 'new_value' WHERE col1 = 10;
```

InnoDB 无法通过索引定位满足 col1 = 10 的行，只能扫描表中的每一行。在扫描的过程中，InnoDB 必须对所有经过的行加锁（即使这些行最终不符合 WHERE 条件），<u>以防止其他事务在扫描期间插入、删除或修改数据。</u>

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
> - 当设置该**参数为 0 时**，事务提交时不会主动触发写入磁盘的操作。
> - （默认）当设置该**参数为 1 时**，表示每次事**务提交时**，都将缓存在 redo log buffer 里的 redo log 直接持久化到磁盘。
> - **参数为 2 时**，表示每次事务提交时，都只是缓存在 redo log buffer 里的 redo log **写到 redo log 文件，注意写入到「 redo log 文件」并不意味着写入到了磁盘**，因为操作系统的文件系统中有个 Page Cache





### redo log 文件满了怎么办

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250126%E4%B8%8B%E5%8D%88101949127.png" alt="image-20250126下午101949127" style="zoom:50%;" />



## bin log

### binlog与redo log区别

- 格式

  - **binlog**

  > 1. **STATEMENT(默认格式)**
  >    - **基于语句的日志记录**：记录实际执行的SQL语句。
  >    - **优点**：<u>日志量小，节省存储空间，尤其适合批量操作</u>。
  >    - **缺点**：某些**<u>非确定性</u>**操作（如 `NOW()`, `RAND()`）可能导致主从不一致。
  >    - **适用场景**：简单、确定性的SQL操作，对数据一致性要求不极端的情况。
  > 2. **ROW**
  >    - **基于行的日志记录**：记录每行数据的变更细节（如修改前后的值）。
  >    - **优点**：精确复制数据变化，确保主从一致性，尤其适合非确定性操作或复杂事务。
  >    - **缺点**：<u>日志量较大，尤其是大范围更新或批量操作时</u>。
  >    - **适用场景**：对数据一致性要求高，或涉及非确定性函数的场景。
  > 3. **MIXED**
  >    - **混合模式**：默认使用`STATEMENT`，但遇到可能导致不一致的操作时自动切换为`ROW`。
  >    - **优点**：在安全性和日志大小之间取得平衡。
  >    - **适用场景**：希望兼顾效率与一致性的通用场景。

  - **redo log**：采用 **物理日志** 格式，记录的是数据页的修改内容

  ```
  # Log record format
  Type: UPDATE
  Page: 0x2A4
  Data: updated data for page 0x2A4 (modified row for id=1)
  ```

  

- 适用对象：binlog是server层概念，redo log是存储引擎层概念

- 写入方式：redolog循环覆盖写，binlog追加写

- 用途：redolog：故障恢复 + 事务持久性， undolog：数据恢复 + 主从复制





### undo log 、redo log、binlog刷盘时机

#### 1. **Undo Log**
- **作用**：支持事务回滚和MVCC（多版本并发控制），记录数据修改前的旧版本。
- **刷盘时机**：
  - Undo log的修改会被记录到**redo log**中，因此其持久化依赖于redo log的刷盘机制。
  - 当事务提交时，若redo log根据配置刷盘（如`innodb_flush_log_at_trx_commit=1`），则undo log的修改也会随redo log一起持久化到磁盘。
  - **注意**：Undo log本身并不直接刷盘，而是通过redo log间接保证其持久性。其空间复用和清理由后台线程管理，与事务提交无直接关联。

---

#### 2. **Redo Log**
- **作用**：确保事务的持久性，记录物理修改，用于崩溃恢复。
- **刷盘时机**：
  - 由参数`innodb_flush_log_at_trx_commit`控制：
    - **1（默认）**：事务提交时，强制将redo log buffer刷到磁盘。
    - **0**：每秒刷盘一次，不保证事务提交时刷盘（可能丢失最近1秒的数据）。
    - **2**：事务提交时仅写入OS缓存，由系统决定刷盘时机（若系统崩溃可能丢失数据）。
  - 后台线程也会定期刷盘，即使事务未提交。

---

#### 3. **Binlog**
- **作用**：记录逻辑操作（如SQL语句或行变更），用于主从复制和数据恢复。
- **刷盘时机**：
  - 由参数`sync_binlog`控制：
    - **0**：依赖文件系统自行刷盘，可能丢失数据。
    - **1（推荐）**：事务提交时强制刷盘，保证数据不丢失。
    - **N**：每N次事务提交后刷盘一次（折中方案，但仍可能丢失N次提交的数据）。

### 两阶段提交

#### 为什么要？解决了什么问题？

  - **问题**：事务提交时，需保证 redo log 和 binlog 的写入完全一致，否则会导致数据丢失或主从不一致。
   - **解决**：通过两阶段提交，将 redo log 和 binlog 的写入绑定为一个原子操作。

| 场景                                                         | 结果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. **先写 redo log，后写 binlog**：<br>若 binlog 写入前崩溃  | - redo log 已提交，但 binlog 缺失<br>**主从数据不一致**（从库丢失事务） |
| 2. **先写 binlog，后写 redo log**：<br>若 redo log 写入前崩溃 | - binlog 已写入，但 redo log 未提交<br>**主库数据丢失，从库多出事务** |

**两阶段提交通过以下逻辑解决这些问题**：

- **崩溃恢复时**：
  1. 检查 redo log 的 `prepare` 状态事务。
  2. 若对应的 binlog 存在且完整，提交事务（重放 redo log）。
  3. 若 binlog 不完整，回滚事务（丢弃 redo log）。

#### 具体步骤？

以 MySQL 单机事务为例（协调 redo log 和 binlog）：

1. **Prepare 阶段（准备阶段）**  
   - InnoDB 将事务的 redo log 标记为 **`prepare` 状态**，并刷盘（根据 `innodb_flush_log_at_trx_commit` 参数决定是否强制刷盘）。
   
   - MySQL Server 将事务的 SQL 操作写入 binlog，并根据 `sync_binlog` 参数决定是否刷盘。若 `sync_binlog=1`，强制刷盘以保证 binlog 持久化。
   
3. **Commit 阶段（提交阶段）**  
   
   - InnoDB 将 redo log 标记为 **`commit` 状态**，事务正式提交。
   - 根据 `innodb_flush_log_at_trx_commit` 参数，可能再次刷盘 redo log。







# 其他

## 将ip地址存入数据库，怎么存放合适

1. **优先使用数据库原生类型**（如PostgreSQL的 `INET`）。

   > **PostgreSQL**：使用 `INET` 或 `CIDR` 类型。
   >
   > ```postgresql
   > CREATE TABLE logs (ip INET);
   > INSERT INTO logs VALUES ('192.168.1.1'), ('2001:db8::1');
   > ```
   >
   > - **优点**：自动校验IP格式，支持网络操作（如 `>>=` 判断子网）。

2. **高频查询且需高性能**：转换为整数或二进制存储。

3. **简单场景**：字符串存储 + 索引优化。

4. **兼容IPv4/IPv6**：用 `VARBINARY(16)` 或分列存储。





## 分库分表

- **水平拆分（Horizontal Sharding）**：将表的行分拆成多个小表，每个小表存储一部分数据。例如，将用户表根据用户ID的范围进行拆分，每个子表存储不同范围的数据。

- **垂直拆分（Vertical Sharding）**：将一个表按照列进行拆分，通常将某些访问频繁的列与不频繁的列分开。适用于大表中的某些列的访问频率与其他列差距较大。



**🌟 常见分库分表的设计策略**

​	•**基于范围的分库分表**：根据某个字段（如ID）进行分库分表，例如将ID为1到1000的数据放在库1中，1001到2000的数据放在库2中。

​	•**基于哈希的分库分表**：对某个字段（如用户ID）进行哈希运算，根据哈希值来决定数据存储在哪个库或表中。

​	• **时间范围分表**：基于时间的范围进行分表，适用于日志表、订单表等按时间生成数据的情况。
