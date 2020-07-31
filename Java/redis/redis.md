[toc]

## Redis

### 1. NoSQL 概述

#### 1.1 为什么使用 NoSQL

数据库演进：

- 90 年代，因为访问量低，使用的是单机数据库
- Memcached ( 缓存 )  + MySQL + 垂直拆分，进行读写分离
- 分库分表 + 水平拆分 + MySQL 集群
- 如今数据类型多且数据量大，MySQL 等关系型数据库表很大，效率较低；需要使用 NoSQL 进行改进

<img src="img/当今网站架构.jpg"  />

#### 1.2 什么是 NoSQL

NoSQL = Not only SQL，泛指非关系型数据库

NoSQL 特点：

- 方便拓展，因为数据之间没有关系
- 高性能
- 数据类型多样，不需要提前设计数据库

大数据的特点：

```markdown
1. 大数据的三V: 描述问题
   - 海量 Volume
   - 多样 Variety
   - 实时 Velocity
2. 大数据的三高: 对程序的要求
   - 高并发
   - 高可拓
   - 高性能
```

#### 1.3 NoSQL 分类

- KV 键值对：Redis，常用于缓存，处理高访问负载
- 文档型数据库：MongoDB，常用数据格式 bson，应用于 Web 应用
- 列存储数据库：HBase，常用于分布式文件系统
- 图形关系数据库：Neo4j，用于设计网络、推荐系统



### 2. Redis 入门

#### 2.1 概述

Redis ( Remote Dictionary Server )，远程字典服务

主要作用：

- 内存存储，断电即失，但 Redis 支持持久化
- 用于高速缓存
- 消息中间件，发布订阅系统
- 地图信息分析
- 计数器，例如微信文章浏览量

#### 2.2 基础知识

- redis 默认有 16 ( 0 - 15 ) 个数据库，默认使用第 0 个

  <img src="img/默认数据库配置.jpg" />

- 基本命令

  ``` markdown
  1. select index ---> 选择数据库
  2. dbsize ---> 当前数据库大小
  3. set key value ---> 设置k-v对
  4. get key ---> 获取key对应的数据
  5. keys * ---> 查看所有key
  6. EXISTS key ---> key是否存在
  7. move key 1 ---> 删除key
  8. flushall/flushdb ---> 清除所有数据/清除当前数据库
  9. EXPIRE key num ---> 设置key的过期时间，单位为s
  10. TTL key ---> 查看key的剩余时间, -1为不过期
  11. type key ---> 查看key的类型
  ```

- Redis 是**单线程**的，因为数据全部存储在内存，故选择单线程是最快的操作，多线程需要上下文切换，并不适合于 CPU 密集场景

  

### 3. 五大数据类型

#### 3.1 String

##### 3.1.1 基础命令

``` markdown
1. APPEND key str ---> 追加字符串，如果key不存在，则新建
2. STRLEN key ---> 获取字符串长度
3. incr/decr key ---> key对应的值自增/自减
4. INCRBY/DECRBY key step ---> key对应的值增加/减少step
5. GETRANGE key start end ---> 类似substr，截取字符串
6. SETRANGE key start end ---> 替换部分字符串
7. setex key timeout value ---> 设置k/v以及过期时间
8. setnx key value ---> 设置k/v，如果key存在则失败
9. mset/msetnx k1 v1 k2 v2 ---> 设置多个k/v，msetnx是原子操作，只能一起成功或者一起失败
```

##### 3.1.2 底层实现

```c
struct sdshdr {
    // buf数组已使用的字节数
    int len;
    // buf数组未使用的字节数
    int free;
    char buf[];
}
```

sdshdr 相比 C 字符串的优势：

- 计数方式不同，C 字符串获取长度时间复杂度 O(n)，

  sdshdr 为 O(1)，返回 len + free

- 杜绝了缓冲区溢出

- 加入了**空间预分配**和**惰性空间释放**

  - 空间预分配：对 sdshdr 拓展时，会分配多余的 free空间 + 1 byte，其余 1 byte 用于存空字符
  - 惰性空间释放：sdshdr 缩减后，不会立即回收多余空间

- 二进制安全，对于二进制文件中存在 '\0' , C 字符串会截断，sdshdr 则不存在这个问题

#### 3.2 List

##### 3.2.1 基本命令

``` markdown
1. LPUSH/RPUSH key value ---> 往列表key头/尾插入数值
2. LPOP/RPOP key ----> 移除列表的头/尾元素
3. LRANGE key start end ---> 获取列表key一部分数据
4. LINDEX key index ---> 获取第index个元素
5. LREM key count value ---> 移除指定个数的value
6. LLEN key ---> 获取列表的长度
7. LTRIM key start stop ---> 通过下标截取指定范围的数据
8. LSET list index value ---> 更新第index元素的value

```

##### 3.2.2 底层实现



