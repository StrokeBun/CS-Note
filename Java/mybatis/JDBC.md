[toc]

## JDBC

### 1. 概述

- JDBC (Java Database Connectivity)是 **独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口**，定义了用来访问数据库的标准Java类库(**java.sql, javax.sql**)，使用这些类库可以以一种**标准**的方法、方便地访问数据库资源。
- 第三方 O/R 工具 Mybatis 等是对 JDBC 的封装

<img src="img/jdbc概述.jpg" style="zoom:80%"/>

### 2. 获取数据库连接

使用配置文件存储信息，获取数据库连接，减少耦合

``` java
        // 1.加载配置文件
        InputStream is = ConnectionTest.class.getClassLoader().getResourceAsStream("jdbc.properties");
        Properties pros = new Properties();
        pros.load(is);

        // 2.读取配置信息
        String url = pros.getProperty("url");
        String driverClass = pros.getProperty("driverClass");

        // 3.加载驱动, 进行设置
        Class.forName(driverClass);
        Properties config = new Properties();
        config.setProperty("user", pros.getProperty("user"));
        config.setProperty("password", pros.getProperty("password"));
        if (pros.getProperty("serverTimezone") != null) {
            config.setProperty("serverTimezone", pros.getProperty("serverTimezone"));
        }
        
        // 4.获取连接
        Connection conn = DriverManager.getConnection(url, config);
        System.out.println(conn);
        conn.close();
```

### 3. CRUD

java.sql 中有 3 个接口定义了对数据库调用

- Statement：用于执行静态 SQL 语句并返回结果
- PreparedStatement：SQL 语句被预编译并存储于此对象中，可以多次高效执行
- CallableStatement：执行 SQL 存储过程

#### 3.1 Statement

- 通过调用 Connection 对象的 createStatement() 方法创建该对象。该对象用于执行静态的 SQL 语句，并且返回执行结果。

  ``` java
  st = conn.createStatement();
  rs = st.executeQuery(sql);
  ```

- Statement 接口中定义了下列方法用于执行 SQL 语句：

  ```java
  int excuteUpdate(String sql); // 执行更新操作
  ResultSet executeQuery(String sql); // 执行查询操作
  ```

- 但是使用Statement操作数据表存在弊端：

  - 存在拼串操作，繁琐
  - 存在 **SQL注入** 问题

#### 3.2 PreparedStament

##### 3.2.1 介绍

- PreparedStatement 接口是 Statement 的子接口，表示一条预编译过的 SQL 语句，通过  preparedStatement(String sql) 方法获取 PreparedStatement 对象
- PreparedStatement 对象所代表的 SQL 语句中的参数用 ? 来表示，调用  setXxx() 方法来设置这些参数；setXxx() 方法有两个参数，第一个参数是参数的索引(从 1 开始)，第二个是参数的值