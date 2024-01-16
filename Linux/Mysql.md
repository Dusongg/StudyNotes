[TOC]



# 1 预备知识

## 1.1 安装

- 获取yum源`rz` :   http://repo.mysql.com/

- 安装yum源：`rpm -ivh mysql57-community-release-e17.rpm`

- 安装mysql：`yum install -y mysql-community-server`

  - 报错：

    ![image-20240111194015333](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240111194015333.png)

    `rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022`

安装完后：

![image-20240111194716623](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240111194716623.png)

- 启动mysql： `start mysqld`

![image-20240111195028586](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240111195028586.png)

- 查服务端口号： `netstat -nltp`

![image-20240111195239301](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240111195239301.png)

## 1.2 登录 & 退出

`mysql -u root -p`

`quit`

## 1.3 配置文件`my.cnf`



# 2 基础知识

## 2.1 链接服务器

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p
```

- `-h`：指明登录部署了mysql服务的主机
- `-P`：指明要访问的端口号
- `-u`：指明登录用户
- `-p`：指明需要输入密码

## 2.2 什么是数据库

- mysql 是数据库服务的客户端，mysqld是数据库服务的服务器端
- mysql本质是基于CS模式的一种网络服务
- 数据库一般指的是再磁盘或内存存储的特定结构的数据

## 2.3 基本使用

### 2.3.1创建表

- 查看数据库文件:

  - ```bash
    cd /var/lib/mysql
    ```

![image-20240112141923131](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112141923131.png)

### 2.3.2 插入数据

![image-20240112142444837](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112142444837.png)

## 2.4 服务器、数据库、表的关系

![image-20240112143620855](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112143620855.png)

 

## 2.5 SQL分类

- DDL（data definition language）数据定义语言，用来维护存储数据的结构
  代表指令: `create, drop, alter`
- DML（data manipulation language）数据操纵语言，用来对数据进行操作
  代表指令： `insert，delete，update`
  - DML中又单独分了一个DQL，数据查询语言，代表指令： select

- DCL（data control language）数据控制语言，主要负责权限管理和事务
  代表指令： `grant，revoke，commit`

## 2.6 存储引擎

- 什么是存储引擎：数据库管理系统如何存储数据，如何为存储的数据建立索引和如何更新、查询数据等技术的实现方法

- `MyISAM` 、 `BDB` 、 `Memory` 、 `innoDB`（最常用）、 `Archive` 、 `NDB`

- ```mysql
  show engines \G；   #查看存储引擎
  ```

![image-20240112145711744](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112145711744.png)

# 3 Mysql数据库的操作

## 3.1 创建和删除

- 创建 —— 本质是在 `/var/lib/mysql`创建一个目录

  - ```mysql
    create [if not exists] database db_name;    #[]里面的是可选项	
    ```

- 删除 —— 本质是在 `/var/lib/mysql`删除一个目录

  - ```mysql
    drop [if exists] database db_name;
    ```



## 3.2 字符集和校验规则

查看编码集（写）：

```mysql
show variables like 'character_set_database';
```

![image-20240112152034459](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112152034459.png)

查看校验集（读）：

```mysql
show variables like 'collation_%';
```

![image-20240112152151872](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112152151872.png)

- 创建一个使用`utf-8`字符集、并带校对规则的数据库（如果没有写，默认为`utf-8`）

```mysql
create database dp_name charset=utf8 collate=utf8_general_ci;
create database dp_name character set utf8 collate utf8_general_ci;   #与  charset=utf8  一样
```



## 3.3 查看数据库

- 查看当前在哪一个数据库中

```mysql
select database();
```

- 查看所有数据库

```mysql
show databases;
```



## 3.4 修改数据库

```mysql
alter database db_name charset=utf8   #对编码格式修改
```



## 3.5 备份和恢复

- 备份

```bash
mysqldump -P3306 -u root -B [数据库名1] [数据库名2] > [数据库备份存储的文件路径]  
```

```mysql
mysqldump -P3306 -u root -B [数据库名] [表名1] [表名2] > [数据库备份存储的文件路径]    #备份同一个数据库中的不同表
```



- 恢复

```mysql
source [数据库备份存储的文件路径]
```

## 3.6 数据库的链接情况

```mysql
show processlist
```



# 4 表的操作

##  4.1 创建表

```mysql
CREATE TABLE [if not exists] table_name (
	field1 datatype comment [文字说明],
	field2 datatype,
	field3 datatype  
)character set [字符集] collate [校验规则] engine [存储引擎];
```

![image-20240112163627454](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112163627454.png)

![image-20240112164028571](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112164028571.png)

## 4.2 查看表

```mysql
desc [表名]
```

```mysql
show create table [表名] \G
```

![image-20240112164545462](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112164545462.png)

## 4.3 修改表

![image-20240112165753877](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112165753877.png)[^1]

[^1]: 下述以该表为例

### 4.3.1重命名

- 改表名

```mysql
alter table [表名] rename to [新表名] 
```

- 改表中某一列的名字

```mysql
alter table [表名] change [old_name] [new_name] [属性]
```

![image-20240112174008991](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112174008991.png)

### 4.3.2 插入数据

```mysql
insert into user values (1, 'yy', '1234', '2003-8-26');
```

### 4.3.3 向表中新增一列

```mysql
alter table [表名] add [新增列名] [列属性] comment [说明] after [列名];
```

![image-20240112170300472](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112170300472.png)

### 4.3.4 修改某一列的属性

```mysql
alter table [表名] modify [列名] [列属性] comment [说明]; 
```

![image-20240112172518036](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112172518036.png)

### 4.3.5 删除某一列

```mysql
alter table [表名] drop [要删除的列名];   
```

![image-20240112172357233](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240112172357233.png)

## 4.4 删除表

```mysql
drop table [表名]   
```



# 5 数据类型

## 5.1 数值类型

`tinyint、 smallint、 mediumint、 int、 bigint` 分别占1、2、3、4、8个字节， 默认为**有符号**整数（无符号：`tinyint unsigned`、 `smallint unsigned`...）

- 和C/C++不同的是，Mysql中，值超出某一类型的数据范围，会直接报错（mysql中，数据类型是一种**约束**）

**![image-20240113115726261](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113115726261.png)**



## 5.2 bit类型

```mysql
[type_name] bit(M)    # M表示位数，范围[1,64], 默认为1
```

## 5.3 浮点数类型

### 5.3.1 `float` /`double`

```mysql
[type_name] float(m, d)   
# m表示数字总个数（与是有符号或无符号无关），d表示小数位数
# 默认有符号 (m - d 表示整数的位数)
# 
```

![image-20240113121523659](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113121523659.png)[^2]

![image-20240113121824043](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113121824043.png)[^3]

[^2]:四舍五入之后超出范围
[^3]:四舍五入之后在合法范围之内



### 5.3.2 `decimal`

- 基本用法与float类似，**精度比`float`高**

![image-20240113125439515](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113125439515.png)

 

## 5.4 字符类型

### 5.4.1 `char`

```mysql
type_name char(L)  
# L为字符的固定长度（一个汉字/字母视为一个字符）
# 最大长度为255
```

### 5.4.2 `varchar`

```mysql
type_name varchar(L) 
# L表示字符长度
# 最大长度为65535字节（65535/3个字符， utf-8一个字符占3个字节）
```

- varchar中用1-3个字节表示内容总长度（“用多少给多少”），所以实际用于数据存储的字节最大有65532 

| 实际存储 | char(4) | varchar(4) | char占用字节 | varchar占用字节 |
| -------- | ------- | ---------- | ------------ | --------------- |
| A        | A       | A          | 4 * 3 = 12   | 1 * 3 + 1 = 4   |



## 5.5 日期类型 

 ![image-20240113133713185](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113133713185.png)

![image-20240113141351919](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113141351919.png)

- 当表被更新时，`timestamp`自动更新成当前时间 

## 5.6 enum 和 set 类型

- enum： 多选一，插入时只能写枚举的常量或者下标（从1开始）
- set ： 多选多，用n个位表示是否选取，1表示选，全0表示空串

![image-20240113144413136](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113144413136.png)建表

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113144437866.png)3表示二进制的011，即插入basketball和football

![image-20240113144500659](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113144500659.png)

![image-20240113144507499](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113144507499.png)

### 5.6.1 enum类型和set类型的选取 —— `find_in_set`

![image-20240113145429969](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113145429969.png)

上述语句选取的是只包含’football‘的，而不是含有’football‘的

- `find_in_set`

```mysql
select find_in_set('a', 'a,b,c');    # 0为false 非0为true（下标）
```

![image-20240113145451901](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113145451901.png)[^4]

[^4]: 查找hobby中包含football的数据

- 若要查询包含多项，可用`and`链接`find_in_set`



# 6 表的约束

- 通过约束，使插入数据库表中的数据如何符合预期

 

## 6.1 空属性约束

```mysql
[type_name] [type] not null
```

- `null`（默认）或 `not null`
- 约束 not null 表示当插入数据时，该列不能为空

![image-20240113172930753](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113172930753.png)

![image-20240113173138486](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113173138486.png)[^5]

[^5]:address没有默认值

![image-20240113173401965](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113173401965.png)[^6]

[^6]:address不能为null



## 6.2 default约束

- 如果同时约束了 `not null` 和 `default` ，在插入数据时可以忽略该列，即填入默认值 
- 没有写默认值，那么默认值默认为`NULL`（没有`not null` 约束时）

```mysql
[type_name] [type] default [defualt_value]   # 设置默认值
```

![image-20240113174236444](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113174236444.png)



## 6.3 comment约束

- `comment`没有实际含义，用来描述字段，类似于注释

```mysql
[type_name] [type] comment [描述]
```



## 6.4 zerofill

- 将数字剩余的宽度填充0

![image-20240113181542927](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113181542927.png)[^7]

[^7]: `int(10) unsigned`中的10，表示因为该无符号整数最大值2^32总共有10位数字（有符号11位，有一位表示符号）



## 6.5 主键 —— `primary key`

- `primary key` 用来约束该字段里面的**数据不能重复**且**不能为空**， 一张表中最多**只能有一个主键**（不代表只能有一列为主键），主键所在的列通常是整数类型

![image-20240113225918430](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113225918430.png)![image-20240113230255081](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113230255081.png)

- 去除主键

```mysql
alter table [table_name] drop primary key; 
```

-  添加主键

```mysql
alter table [table_name] add primary key([某一字段名])
```

- 复合主键（当多个字段值**均相同**时才报错）

![image-20240113232251799](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240113232251799.png)
