[toc]



## Spring 整合持久层

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

  

