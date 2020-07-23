[toc]

## Spring 5

### 1. 引言

#### 1.1 EJB 的问题

Spring 是用于**改善 EJB 的问题**

EJB 的问题

- **运行环境苛刻**，EJB 必须依靠 EJB 容器实现，需要运行在 Weblogic 等 Application 服务器上，这些服务器都是收费的，且未开源
- **代码移植性差**，EJB 应用必须实现 Weblogic 等服务器的相关接口，更改服务器后代码必须重写

#### 1.2 Spring介绍

Spring 是一个轻量级的 Java EE 解决方案，并整合众多优秀的设计模式

- 轻量级：运行环境没有额外要求，可运行在支持 Servelt 引擎的服务器；代码移植性高，不需要额外实现接口
- Java EE 解决方案：Spring 可实现 Controller、Service、DAO 各层

#### 1.3 工厂设计模式

<font color=blue>Spring 的本质即使用工厂，ApplicationContext(applicationContext.xml)</font>

#####  1.3.1 简单反射工厂

通过反射来实现工厂的建造返回

``` java
Class clazz = Class.forName("xxx.xx.xx.UserServiceImpl");
UserService userService = (UserService)clazz.newInstance(); // newInstance调用了无参构造函数
return userService;
```

上述实现仍需要 UserServiceImpl 的全限定名，转为文件来设置全限定名，通过读取配置文件设置

``` properties
# xxx.properties
userService = xxx.xx.xx.UserServiceImpl
```

``` java
// Properties集合来存储 properties 文件的内容
// Properties是一个<String, String>的特殊Map
public class BeanFactory {
    private static Prorerties env = new Prorerties();
    // 使用静态代码块来读入配置文件
    static {
        InputStream inputStream = BeanFactory.class.
            getResourceAsStream("xxx.properties");
        env.load(inputStream); // 读取输入流
    }
    
    public static UserService getUserService() {
        Class clazz = Class.forName(env.getProperty("userService"));
		return (UserService)clazz.newInstanwence();
    }
}
```

只需要改动配置文件即可，不需要重新编译

##### 1.3.2 通用工厂设计

仅使用简单工厂将导致工厂数量大大增加，并且工厂实现代码存在大量冗余，设计通用工厂来实现

``` java
public class BeanFactory {
    private static Prorerties env = new Prorerties();
    // 使用静态代码块来读入配置文件
    static {
        InputStream inputStream = BeanFactory.class.
            getResourceAsStream("xxx.properties");
        env.load(inputStream); // 读取输入流
    }
    
    public static Object getBean(String key) {
        Object ret = null;
        try {
            Class clazz = Class.forName(env.getProperty(key));
        } catch(Exception e) {
            ...
        }
		return ret;
    }
}
```

### 2. 第一个 Spring 程序

#### 2.1 环境搭建

Spring 的配置文件

``` markdown
1. 配置文件的存放位置: 任意, 但需要进行配置文件路径的配置
2. 配置文件的命名: 任意，推荐 applicationContext.xml
```

#### 2.2 Spring 核心 API

- ApplicationContext：创建对象的工厂

  ``` markdown
  1. 接口类型
  非web环境接口实现: ClassPathXmlApplicationContext
  web环境接口实现: XmlWebApplicationContext
  2. 重量级资源
  一个应用程序只会创建一个工厂对象，线程安全
  ```

#### 2.3 程序开发

- 程序开发流程

  ``` markdown
  1. 配置文件
  <!-- id:唯一标识； class:类的全限定名 -->
  <bean id="person" class="com.stroke.demo.entity.Person"></bean>
  2. 通过工厂类获得对象
  ```

- 代码实现

  ```java
  // 1. 获得Spring的工厂
  ApplicationContext ctx = 
      new ClassPathXmlApplicationContext("/applicationContext.xml");
  // 2. 创建对象,下述为几种实现
  // getBean通过反射创建对象，调用该类的无参构造函数，如果不存在无参构造将报错
  Person person = (Person) ctx.getBean("person");
  Person person = ctx.getBean("person", Person.class);
  Person person = ctx.getBean(Person.class); // 该实现要求配置文件中只有一个bean的class为Person，否则抛出NoUniqueBeanDefinitionException异常
  ```

- 配置文件的细节

  ``` markdown
  1. 只配置 class 的情况下
  a) Spring 会自动生成 id 值，默认为 类的全限定名#数字
  b) 当 bean 只使用一次的时候，才能省略 id
  2. name 属性
  作用: 为 bean 对象定义别名，使用类似 id
  与 id 的区别:
  	a) name 可以有多个，id 唯一
  	b) containsBeanDefinition 方法只能判断 id 不能判断 name，containsBean 方法二者皆可        判断
  ```

  #### 2.4 Spring 工厂的底层实现(简易版)

  Spring 工厂可以通过调用**私有**的构造函数创建对象，反射面前无私有

  <img src="img/spring工厂简单流程.jpg" alt="avatar" style="zoom:70%">

### 3. 注入

#### 3.1 简介

注入即通过工厂与配置文件，为创建对象的**成员变量赋值**，去除编码方式赋值存在的耦合

#### 3.2 如何注入

- 成员变量提供 setter

- 配置文件

  ``` properties
      <bean id="person" class="com.stroke.demo.entity.Person">
          <property name="id">
              <value>1</value>
          </property>
          <property name="name">
              <value>bzzb</value>
          </property>
      </bean>
  ```

#### 3.3 Set 注入详解

##### 3.3.1 JDK内置类型

**1. String + 基本类型**

``` properties
<value>要注入的值</value>
```

简化写法

``` properties
<property name="id" value="1" />
<property name="name" value="bzzb" />
```

**2. List、Set等容器**

``` properties
<list>
	<value>注入的值1</value>
	<value>注入的值2</value>
	<value>注入的值3</value>
</list>
```

**3. Map**

``` properties
<map>
	<entry>
		<key><value> uzi </value></key>
		<value> yyds </value>
	</entry>
</map>
```

##### 3.3.2 自定义类型

**1. 第一种方式**

配置文件

``` properties
<bean id="userService" class="xxx.UserServiceImpl">
	<property name="userDao">
		<bean class="xxx.UserDaoImpl" />
	</property>
</bean>
```

**2. 第二种方式**

第一种方式存在的问题：

- 当 Dao 被多个 Service 调用时，存在多个 bean 的相同配置，代码冗余
- 创建了多个 Dao 实例，实际上可以使用同一个

使用引用改善

``` properties
<bean id="userService" class="xxx.UserServiceImpl">
	<property name="userDao">
		<ref bean="userDAO" />
	</property>
</bean>

<!-- 简化写法 -->
<bean id="userService" class="xxx.UserServiceImpl">
	<property name="userDao" ref="userDAO" />
</bean>
```



#### 3.4 构造注入

通过调用**有参构造**注入属性

配置文件

``` properties
<bean id="customer" class="com.stroke.demo.entity.Customer">
     <constructor-arg value="bzzb"></constructor-arg>
     <constructor-arg value="22"></constructor-arg>
</bean>
```

如何**区分重载**的构造函数？

**1. 参数个数不同**

``` properties
控制 constructor-arg 的个数
```

**2. ** **参数个数相同**

``` properties
通过 type 指定类型
<bean id="customer" class="com.stroke.demo.entity.Customer">
     <constructor-arg type="int" value="22"></constructor-arg>
</bean>
```



