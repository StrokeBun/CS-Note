## Spring面试题

#### 1. Spring Bean的作用域

- singleton：默认值，单例模式
- prototype：原型模式，每次创建一个新实例
- request：每次 HTTP 请求创建一个实例，仅用于 WebApplicationContext
- session：同一个 HTTP Session 共享一个实例，仅用于 WebApplicationContext
- gloabalSession：用于 Portlet 应用，不常使用，仅用于 WebApplicationContext



#### 2. BeanFactory和FactoryBean

- BeanFactory：是个接口，定义 IOC 容器或对象工厂基本形式，实现该接口的有 ApplicationContext、XmlBeanFactory 等；
- FactoryBean：是一个工厂 bean，可以自定义 bean 的创建过程，为 bean 提供了灵活处理的方式



#### 3. Bean的线程安全性

- 对于 prototype，线程不存在共享，是线程安全的
- 对于 singleton，线程存在共享。如果是无状态 bean（例如 Service，DAO），则是线程安全的；如果是有状态的，则是不安全的，可以使用 ThreadLocal 为每个线程存储独立的副本，实现线程安全



#### 4. Spring 如何解决循环依赖问题

循环依赖问题是指 A 类中引入 B 类属性，B 类中引入 A 类属性（或者更多类形成循环），由于 Spring 利用反射机制进行注入，循环依赖将导致实例无法生成。

#### 4.1 循环依赖场景

- 单例 setter 注入，能解决
- 多例 setter 注入，无法解决
- 构造器注入，无法解决
- 单例代理对象 setter 注入，有可能解决
- DependsOn 循环依赖，无法解决

#### 4.2 解决循环依赖

Spring 的三级缓存

- 一级缓存 Map<String, Object> singletonObjects：存储已经创建（实例化、注入、初始化）完毕的单例 Bean 实例
- 二级缓存 Map<String, Object> earlySingletonObjects：存储已经实例化的的 Bean 实例
- 三级缓存 Map<String, ObjectFactory<?>> singletonFactories：存储该 Bean 提前暴露的引用，

解决循环依赖的过程如下图

<img src="img/spring解决循环依赖.jpg">

分析：

- 仅使用一级缓存：完全无法解决循环依赖问题

- 仅使用二级缓存：可以解决部分循环依赖问题，如果该 Bean 实现 AOP 则会产生问题

  <img src="img/带AOP循环依赖的二级缓存方案.jpg" style="zoom:40%">

  如上图所示，实现了 AOP 的 Bean 在初始化之后经过 AOP 生成代理类，但该代理类和 B 类持有的 A 类不是一个对象。

- 三级缓存：三级缓存中存放 bean 对应的工厂 ObjectFactory，是个 Lambda 表达式

  ``` java
  addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
  ```

  earlyBeanReference 取得该 bean 的代理对象，后续再将代理对象放入二级缓存。