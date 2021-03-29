## InnoDB

### 1. 介绍

InnoDB 是默认的存储引擎，以 **页** 作为磁盘和内存交互的基本单位，一次至少读写一个页，默认页大小为 16 KB

关键特性：

- 插入缓冲：
- 两次写：刷新脏页时，先复制到 doublewrite buffer，再分两次写到磁盘上；写入崩溃时可以从 doublewrite buffer 获取副本进行恢复；
- 自适应哈希索引：为热点页创建哈希索引，提高查询速度；
- 异步 IO
- 刷新邻接页：刷新脏页时，检测页所在区的所有页，如果是脏页则一同刷新



### 2.行格式

InnoDB 具有 4 种行格式，分别是 `Compact`、`Redundant`、`Dynamic` 和`Compressed`

#### 2.1 Compact

`Compact` 的行格式如下图所示

<img src="img/compact行格式.jpg" style="zoom:40%">

##### 2.1.1 额外信息

- 变长字段长度：按照列的顺序 **逆序** 存储带变成字段的列实际占用的字节数
- NULL 值列表：使用位图逆序存储各列是否为 NULL
- 记录头信息：记录该行的相关信息

##### 2.1.2 真实数据

真实数据部分存储实际数据，并含有三个隐藏列

- `row_id` ：行 id，非必需，如果未自定义主键，则默认生成 `row_id` 作为隐藏主键
- `transaction_id`：事务 id，MySQL 生成，实现事务
- `roll_pointer` ：回滚指针，MySQL 生成，实现事务回滚 

行溢出：当某一列的数据特别多时，在 `真实数据` 部分只存储一部分数据，把剩余的数据存储在其他页中（存储这些数据的页面称为 `溢出页` ），用一定数量字节存储这些页的地址。

#### 2.2 Dynamic

`Dynamic` 是 InnoDB 默认的行格式，基本与 `Compact` 相同，但在处理行溢出上不存储部分数据，而是将所有数据都存储在溢出页。

<img src="img/dynamic行溢出.jpg" style="zoom:30%"> 

#### 2.3 Compressed

`Compressed` 会采用压缩算法对页面进行压缩，以节省空间，其余与 `Dynamic`相似。

#### 2.4 Redundant

`Redundant` 是 5.0 版本前的默认格式，不多介绍。



### 3. 数据页结构

InnoDB 的数据页结构如下图所示

<img src="img/innodb数据页结构图.jpg" style="zoom:70%" >

页默认大小为 16 KB，分为 7 个部分：

- File Header：文件头，记录页的通用信息，保存了上一个页和下一个页的页编号，形成页链表（B+ 树的叶节点）

- Page Header：表示页的专有信息

- Infimum + Supremum：页中数据形成链表，Infimum、Supremum 分别是链表的头、尾哨兵节点

- User Records：存储数据行，插入新数据则从 Free Space 中分配空间；行的记录头信息中包含 `next_record` 将各数据行组成链表，`delete_mask` 记录该行是否被删除（采用懒删除，如果插入新数据可能复用该行），下图为 User Records 数据存储示意

  <img src="img/页中数据示意.jpg" style="zoom:50%">

- Free Space：页中未使用的部分

- Page Directory：页中 `槽` 的相对位置，InnoDB 会把页中的记录划分为若干个组，每个组的最后一个记录的地址偏移量作为一个 `槽`，查找时先进行二分定位槽，再遍历查找行，加快查找速度

- File Trailer：同步校验部分



