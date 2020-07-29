[toc]



## Spring 事务

#### 1. Spring 整合 Mybatis

#### 1.1 Mybatis 开发步骤

- 实体
- 实体别名
- 表
- 创建 DAO 接口
- 实现 Mapper 文件
- 注册 Mapper 文件
- Mybatis API 调用

#### 1.2 Mybatis 的痛点

配置繁琐、代码冗余

 #### 1.3 整合思路

使用 SqlSessionFactoryBean 封装下述代码

``` java
InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

使用 MapperScannerConfigure 封装下述代码，并约定 DAO 对象的 id 值为 **接口首字母小写**

``` java
SqlSession session = sqlSessionFactory.openSession();
UserDAO userDAO = session.getMapper(UserDAO.class);
```

#### 1.4 整合步骤

待续。。。。

### 2. Spring 的事务处理

#### 2.1 什么是事务

事务即保证业务完整性的一种数据库机制，特点：ACID 

#### 2.2 如何控制事务

- JDBC

  ```java
  Connection.setAutoCommit(false); // 开启事务
  Connection.commit(); // 提交
  Connection.rollback(); // 回滚
  ```

- Mybatis：默认开启事务，底层还是 Connection

  ``` java
  sqlSession.commit();
  sqlSession.rollback();
  ```

#### 2.3 Spring 事务开发

Spring 通过 AOP 的方式进行事务开发

- 原始对象

  ``` java
  // DAO 作为 Service 的成员变量，依赖注入
  public class XXXServiceImpl {
      private XXXDAO xxxDAO;
      setter 方法
  }
  ```

  ``` java
  public class UserServiceImpl implements UserSerivce {
      private UserDAO userDAO;
  
      public UserDAO getUserDAO() {
          return userDAO;
      }
  
      public void setUserDAO(UserDAO userDAO) {
          this.userDAO = userDAO;
      }
  
      @Override
      public void register() {
          userDAO.register();
      }
  }
  ```

  ``` xml
      <bean id="userService" class="com.stroke.demo.service.UserServiceImpl">
          <property name="userDAO" ref="userDao" />
      </bean>
  ```

  

- 额外功能：Service 调用前开启事务，调用后提交事务，抛出异常则进行回滚

  Spring 通过 DataSourceTransactionManager 对上述进行了封装，需要注入 DataSource

  ``` xml
      <bean id="dataSourceTransactionManager"class="org.springframework.jdbc.
                                               dataSource.DataSourceTransactionManager">
          <property name="dataSource" ref="dataSource"/>
      </bean>
  ```

- 切入点

  ``` markdown
  @Transactional
  该注解声明业务
  1. 用于类: 该类所有方法都会加入事务
  2. 用于方法: 该方法加入事务
  ```

- 组装

  ``` xml
  <!--该标签开启注解扫描-->
  <tx:annotation-driven transaction-manager="dataSourceTransactionManager" />
  ```


### 3. 事务属性

#### 3.1 介绍

事务属性是描述事务特征的一系列值，包括

- 隔离属性
- 传播属性
- 只读属性
- 超时属性
- 异常属性

#### 3.2 如何添加事务属性

在 @Transactional 注解中直接添加

``` java
@Transactional(isloation=, propagation=, readOnly=...)
```

#### 3.3 隔离属性 ISOLATION

隔离属性：描述事务解决**并发问题**的特征

并发：多个事务同时访问操作相同数据，会产生脏读、不可重复读、幻读问题

##### 3.3.1 脏读

脏读：**<font color=blue>一个事务，读取了另一个事务未提交的数据</font>**

解决方案：设置隔离属性为读取已提交的

``` java
@Transactional(isloation=Isolation.READ_COMMITED)
```

##### 3.3.2 不可重复读

不可重复读：**<font color=blue>在一个事务中多次查询同一个数据的结果不同</font>**，与脏读的差别是其他事务已经进行了提交

解决方案：设置为重复读，底层是数据库对该行数据加了 **行锁**

``` java
@Transactional(isloation=Isolation.REPETABLE_READ)
```

##### 3.3.3 幻影读

幻影读：**<font color=blue>在一个事务中多次查询<font color=red>整表</font>的结果不同</font>**，与不可重复读的差别是查询整表的数据

解决方案：设置为序列化，底层是数据库加了 **表锁**

``` java
@Transactional(isloation=Isolation.SERIALIZABLE)
```

##### 3.3.4 总结

并发安全：SERIALIZABLE > REPETABLE_READ >READ_COMMITED，效率反之

数据库对隔离属性的支持

| 隔离属性       | MySQL | Oracle |
| :------------- | :---- | ------ |
| READ_COMMITED  | ✔     | ✔      |
| REPETABLE_READ | ✔     | ×      |
| SERIALIZABLE   | ✔     | ✔      |

Oracle 采用**版本控制**来实现重复读问题

##### 3.3.5 默认隔离属性

``` markdown
ISOLATION_DEFAULT: 根据调用的数据库设置默认隔离属性

MySQL: REPETABLE_READ，可使用 select @@transaction_isolation 查看
Oracle: READ_COMMITED
```

#### 3.4 传播属性

隔离属性：描述事务解决**嵌套问题**的特征

事务嵌套：即一个事务包含多个小事务，例如 service 调用另一个 service 的情况

产生的问题：小事务导致大事务失去了 **原子性**

``` java
// 使用方法
@Transactional(propagation=Propagation.传播属性)
```

- 传播属性的值与用法

  | 传播属性的值  | 外部不存在事务 | 外部存在事务               | 常用场景 |
  | ------------- | -------------- | -------------------------- | -------- |
  | REQUIRED      | 开启新事务     | 融合到外部事务             | 增删改   |
  | SUPPORTS      | 不开启新事务   | 融合到外部事务             | 查询     |
  | REQUIRES_NEW  | 开启新事务     | 挂起外部事务，创建新的事务 | 日志记录 |
  | NOT_SUPPORTED | 不开启新事务   | 挂起外部事务               |          |
  | NEVER         | 不开启新事务   | 抛出异常                   |          |
  | MANDATORY     | 抛出异常       | 融合到外部事务             |          |

- 默认传播属性：REQUIRED

#### 3.5 只读属性 

只读属性：针对于查询操作的业务添加只读属性，提高运行效率

``` java
// 使用方法
@Transactional(readOnly = true) // 默认是 false
```

#### 3.6 超时属性

超时属性：指定事务的最长等待时间

等待原因：访问的数据被其他数据加了锁

``` java
// 使用方法
@Transactional(timeout = 2) // 单位是秒，默认值是-1，具体时间依赖于数据库
```

#### 3.7 异常属性

Spring 默认对于 RuntimeException 及其子类 采用回滚的策略

``` java
@Transactional(rollbackFor = {java.lang.Exception.class}, 
               noRollbackFor={java.lang.RuntimeException.class})
// rollbackFor 指定回滚的异常，noRollbackFor 指定不回滚的异常
// 建议: 实战中使用 RuntimeException 及其子类 使用异常属性的默认值
```

