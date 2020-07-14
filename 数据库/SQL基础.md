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
INSERT INTO Product VALUES ('0001', 'T恤衫', '衣服'), ('0002', '打孔器', '办公用品');

-- 从OtherProduct复制
INSERT INTO Product SELECT * FROM OtherProduct;
```

#### 5.2 删

``` sql
-- DELETE 只能使用 WHERE 子句
DELETE FROM Product WHERE product_id = 1;
-- TRUNCATE 删除表中所有数据
TRUNCATE Product;
```

#### 5.3 改

``` sql
UPDATE Product SET product_price = 500 WHERE product_id = 1;
```

#### 5.4 查

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

##### 5.4.4 运算符

``` sql
>, <, = -- 对应大于，小于，等于
<> --不等于
-- IS NULL 判断是否为空
SELECT * FROM Product WHERE product_price IS NULL;
```

#### <font color=red>5.5 事务</font>

**事务**：在同一个处理单元中执行的一系列更新处理的集合；MySQL默认一条SQL语句就是一个事务

``` sql
-- MySQL的事务实现
START TRANSACTION; -- 开启事务
INSERT INTO Product VALUES ('0001', 'T恤衫', '衣服'), ('0002', '打孔器', '办公用品');
COMMIT; -- 提交

-- 事务回滚
START TRANSACTION;
INSERT INTO Product VALUES ('0001', 'T恤衫', '衣服'), ('0002', '打孔器', '办公用品');
ROLLBACK; -- 回滚
```

事务的 **ACID**特性：

- **Atomicity**：原子性，事务要么全部执行，要么全部不执行
- **Consistency**：一致性，事务前后数据的完整性必须保持一致
- **Isolation**：隔离性，不同事务之间互不干扰
- **Durability**：持久性，事务一旦提交，数据改变就是永久性的

### 6. 聚合与排序

#### 6.1 聚合

聚合，即将多行汇总与一行，用于汇总的函数称为**聚合函数**

``` sql
-- 常用聚合函数,其中p为列字段
COUNT(p) -- 计算该字段非NULL的行数
COUNT(*) -- 会得到实际行数，即使全为NULL也计算
SUM(p) -- 计算该字段的和，不会计算NULL
AVG(p) -- 计算该字段的平均值，不会计算NULL
MAX(p) -- 最大值
MIN(p) -- 最小值 
```

#### 6.2 分组

``` sql
-- GROUP BY 用于切分表
SELECT product_type FROM Product GROUP BY product_type;
-- HAVING 用于过滤分组，用法与 WHERE 相同
SELECT product_type, COUNT(*) AS orders 
FROM Product GROUP BY product_type HAVING COUNT(*) >= 2;
```

GROUP BY 的注意事项

- 只能用于 SELECT 中，写在 WHERE/FROM 之后
- 不能使用别名
- GROUP BY 聚合结果无序
- WHERE 子句不能使用聚合函数

#### 6.3 排序

``` sql
-- 使用 ORDER BY进行排序，默认升序
SELECT * FROM Product ORDER BY product_id;
-- DESC 进行降序
SELECT * FROM Product ORDER BY product_id DESC;
```



### 7. 函数、谓词、CASE表达式

#### 7.1 函数

**算术函数**

``` sql
-- 算术函数
+ - * /
ABS(num) -- 绝对值函数
MOD(被除数，除数) -- 取余
ROUND(对象数值，保留小数的位数) -- 对该数进行四舍五入
```

**字符串函数**

``` sql
CONCAT(str1, str2, str3) -- 连接字符串
LENGTH(str) -- 字符串长度
LOWER(str) -- 转为小写
UPPER(str) -- 转为大写
REPLACE(str1, str2, str3) -- 字符串替换
-- REPLACE('ABC12','ABC', 'abc') = 'abc12'
SUBSTRING(str FROM 截取的起始位置 FOR 截取的字符数)
```

**日期函数**

``` sql
CURRENT_DATE
CURRENT_TIME
CURRENT_TIMESTAMP
```

#### 7.2 谓词

**LIKE**：用以模糊搜索

``` sql
-- %用于匹配任意字符
SELECT * FROM Product WHERE product_name LIKE '%外套%'
-- _用于匹配一个字符
SELECT * FROM Product WHERE product_name LIKE '_外套' -- 匹配大外套
```



### 8. 联结

``` sql
-- 等值联结
SELECT vend_name, prod_name, prod_price
FROM vendors AS v, products AS p
WHERE v.vend_id = p.vend_id;
-- 内部联结
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;
-- 外部联结，包括了相关表中没有联结的行，例如vendors中有id为4，但products中无为4的id
-- 使用内部联结将无法查询，而使用外部联结将以NULL进行填充
SELECT vend_name, prod_name, prod_price
FROM vendors LEFT OUTER JOIN products
ON vendors.vend_id = products.vend_id;
```



