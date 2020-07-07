[toc]

## SQL 基础

### 1. 数据库介绍

**关系型数据库**：采用关系模型(即二维表格)组织数据的数据库，特点是**以行为单位进行读写**



### 2. 数据库操作基础指令

#### 2.1 创建数据库

``` sql
# 注释1
CREATE DATABASE Shop; -- 注释2
/*
注释3
*/
```

#### 2.2 删除数据库

``` sql
DROP DATABASE Shop;
```



### 3. 基本数据类型

这里只介绍最基础的 4 种类型

- **INTEGER型**：整型
- **CHAR型**：定长字符串，可在括号中指定字符串的最大长度，存储大小固定为最大长度
- **VARCHAR型**：可变长字符串，存储大小为字符串实际长度
- **DATE型**：存储日期



### 4. 表操作基础指令

#### 4.1 创建表

``` sql
CREATE TABLE Product
( product_id CHAR(4) NOT NULL,
  product_name VARCHAR(100) NOT NULL,
  product_type VARCHAR(32) NOT NULL
);
```

#### 4.2 删除表

``` sql
DROP TABLE Product;
```

#### 4.3 表定义更新

``` sql
-- 添加列
ALTER TABLE Product ADD COLUMN product_price INT(20) NOT NULL;
-- 删除列
ALTER TABLE Product DROP COLUMN product_price;
-- 更改表名
RENAME TABLE Poduct to Product;
```



### 5. 数据操作

#### 5.1 增

``` sql
START TRANSACTION; -- 开启事务
INSERT INTO Product VALUES ('0001', 'T恤衫', '衣服');
INSERT INTO Product VALUES ('0002', '打孔器', '办公用品');
INSERT INTO Product VALUES ('0003', '运动T恤', '衣服');
INSERT INTO Product VALUES ('0004', '菜刀', '厨房用具');
COMMIT; -- 提交
```



### 5.4 查

##### 5.4.1 基本查询

``` sql
-- 基本查询
SELECT product_id, product_name, product_type FROM Product;
-- 设定别名
SELECT product_id AS id, 
       product_name AS name, 
       product_type AS type 
FROM Product;
```

##### 5.4.2 DISTINCT

使用 DISTINCT 删除重复行, DISTINCT 只能用于**第一个列名**之前

``` sql
SELECT DISTINCT product_type FROM Product;
```

##### 5.4.3 WHERE

WHERE 会先查询符合条件的的记录

```sql
-- 基本使用
SELECT product_name, product_type FROM Product
WHERE product_type = '衣服';
```



