[toc]

## Spring 注解编程

### 1. 基本概念

注解编程即在类或者方法上加入注解，完成特定功能的开发

#### 1.1 注解编程的原因

- 开发方便，代码简洁
- Spring 开发的潮流

#### 1.2 注解的作用

- 替换 XML，简化配置
- 替换接口，实现调用双方的契约性

#### 1.3 使用注解还能否解耦？

可以，通过 Spring 配置文件能够覆盖注解，来实现解耦



### 2.  Spring 的基础注解

Spring 2.x 开始支持的注解

#### 2.1 对象创建注解

首先需要开启注解扫描

``` xml
<context:componet-scan base-package:"xxx.xxx" />
```

##### 2.1.1 @component

作用：替换配置文件中的 bean 标签，id 默认值为 **类名首字母小写**

**细节**

- 指定工厂创建对象的 id 值

  ``` java
  @component("user")
  ```

- 覆盖注解时，配置文件的 id 值需要与注解的相同

  ``` xml
  <bean id="user" class="xxx.xxx" />
  ```

@component 的衍生注解

``` markdown
@Repository  ---> xxxDAO
@Service     ---> xxxService
@Controller  ---> xxxController

1. 这些衍生注解就是 @component, 作用、用法、用法一致
2. 提供衍生注解的作用: 更加准确表达一个类型的作用
```

##### 2.1.2 @Scope

作用：替换配置文件中的 scope 属性

``` java
// 如果没提供@Scope, 默认为单例
// 单例/原型 对应 singleton/prototype
@Scope("singleton")
```

##### 2.1.3 @Lazy

作用：替换配置文件中的 lazy 属性

``` java
@Lazy // 使用该注解，Spring 将会延迟创建
```

##### 2.1.4 生命周期方法相关注解

```markdown
1. 初始化方法 @PostConstruct
	InitializingBean
2. 销毁方法 @PreDestroy
	DisposableBean
	
注意: 上述注解不是 Spring 提供, 而是 JavaEE 的规范
	
```



#### 2.2 注入相关注释

##### 2.2.1 @Autowired

``` java
@Autowired
private UserDAO userDAO;
```

- 默认基于类型注入，注入的类必须是目标成员类型或者子类

- @Autowired + @Qualifier 可以实现基于 id 注入

  ``` java
  @Autowired
  @Qualifer("userDAOImpl")
  private UserDAO userDao;
  ```

- @Autowired 的放置位置

  - setter
  - 放在成员变量上，使用反射实现赋值

- Java EE 规范

  - @Resource，效果类似 @Autowired + @Qualifier
  - @Inject，与 @Autowired 相同，但在 Spring 开发中不使用

##### 2.2.2 @PropertySource

用来定位 .properties 文件

``` java
@PropertySource("classpath:xxx.properties")
// 用以替换以下标签
<context:property-placeholder location="classpath:/xxx.properties" />
```

##### 2.2.3 @Value

- @Value 不能用以静态成员变量
- @Value + .properties 不能注入集合类型



#### 3. 对于注解开发的思考

- 配置互通

  ```markdown
  注解和配置文件可以配合使用
  ```

- 什么时候使用注解

  ``` markdown
  1. 基础注解用于程序员的开发过程，使用在自己编写的类上
  2. 第三方代码类仍需要使用配置文件
  ```

### 4. Spring 的高级注解

#### 4.1 配置 Bean

Spring 3.x 提供的注解，用于替换 xml 文件，本质上也是 @Component 的衍生注解

``` java
@Configuration
public class AppConfig {
    
}
```

``` java
// 创建工厂
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
```





