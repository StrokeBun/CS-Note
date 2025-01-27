## MySQL 锁

### 1. 锁分类

从对数据操作的粒度可分为

- 全局锁：锁住整个数据库，一般用于全库逻辑备份

- 表锁：锁定整个表
- 行锁：锁定当前行

从对数据操作的类型分：

- 读锁：共享锁，针对同一数据，多个读操作可以同时进行而不互相影响
- 写锁：排它锁，当前写操作未完成前，阻断其他写锁和读锁



### 2. MySQL 锁

#### 2.1 概述

MySQL 对于不同存储引擎支持不同的锁机制

| 存储引擎 | 表锁 | 行锁 | 页面锁 |
| -------- | ---- | ---- | ------ |
| MyISAM   | 支持 |      |        |
| InnoDB   | 支持 | 支持 |        |
| MEMORY   | 支持 |      |        |
| BDB      | 支持 |      | 支持   |

三种锁的特性

|<span style="display:inline-block;width:50px">锁类型</span>| 特点                                                         |
| ------  | ------------------------------------------------------------ |
| 表锁    | 偏向 MyISAM 存储引擎，开销小，加锁快，不会出现死锁；但锁粒度大，冲突概率高，并发度低 |
| 行锁    | 偏向 InnoDB 存储引擎，开销大，加锁慢，会产生死锁；锁粒度小，冲突概率小，并发度高 |
| 页面锁  | 性能介于行锁与表锁之间，会产生死锁                           |



#### 2.2 MyISAM 表锁

MyISAM 存储引擎只支持表锁，在执行 SELECT 语句时，会给涉及的所有表加读锁，对于更新操作则添加写锁

- 读锁：不会阻塞其他用户对该表的读请求，但会阻塞写请求
- 写锁：阻塞其他用户的读写请求

MyISAM 的读写调度是以写优先，因此大量的更新将使查询操作阻塞



#### 2.3 InnoDB 行锁

##### 2.3.1 行锁类型

InnoDB 行锁类型：

- `LOCK_REC_NOT_GAP` : 锁住特定行
- `Gap Locks` ：间隙锁，当用范围条件请求数据时，InnoDB 会给符合条件的数据加锁，对于在该范围但是不存在的数据，这些数据会被加锁，在锁释放之前无法插入
- `LOCK_ORDINARY/Next-Key Lock`：是 `LOCK_REC_NOT_GAP` 和  `Gap Locks` 的结合，锁住当前行和之前的间隙
- `Insert Intention Locks`：当插入时碰到间隙锁，则会生成该锁

##### 2.3.2 加锁规则

在默认的 可重复读 的隔离等级下， 加锁有如下规则：

- 原则1：加锁基本单位是 `next-key lock`
- 原则2：查找过程中访问到的行才会加锁
- 优化1：唯一索引的等值查询，`next-key lock` 退化为行锁
- 优化2：唯一索引的范围查询访问到不满足条件的第一行
- 优化3：非唯一索引的等值查询，向右遍历时且最后一个值不满足等值条件的时候，`next-key lock` 退化为间隙锁



##### 2.3.3 两阶段提交与死锁

InnoDB 的行锁使用了 **两阶段协议**，事务提交之后才会释放锁。故在事务中把最可能冲突的锁尽量往后放，减少锁持有的时间和事务之间的锁等待，提升并发度。

由于事务提交之后才释放锁，故可能产生死锁问题，举例如下

<img src="img/行锁死锁示意图.jpg" style="zoom:40%">