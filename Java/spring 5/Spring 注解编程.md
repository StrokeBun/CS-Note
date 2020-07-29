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









