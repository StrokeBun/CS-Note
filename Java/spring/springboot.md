[toc]

## Springboot

### 1.介绍

主要用于解决 Spring 过于繁琐的配置，"约定大于配置"

### 2. 注解

#### 2.1 常用注解

**@RestController**: 将返回的对象数据以 JSON 形式写入 HTTP Response

**@RestController** 和 **@Controller**：

- <font color = blue>@RestController = @Controller + @ResponseBody</font>
- @Controller 返回一个页面，常用于传统的 MVC 开发；@RestController 返回 JSON 或者 XML 数据，常用于前后端分离



**@RequestMapping**: 指定映射url，可以通过 method 指定方法(GET、POST)

**@PathVariable**: 取 url 地址中的参数

**@RequestParam**: url 的查询参数值

**@RequestBody**: 将 HTTP 请求 body 中的 JSON 数据反序列化为 Java 对象



#### 2.2 @PostConstruct 和 PreDestory

这两个注解作用于 Servlet 生命周期，被这两个注解修饰的方法在整个 Servlet 生命周期**只执行一次**

**作用**:

- @PostConstruct：项目启动前执行，一般用于执行初始化操作，在**构造函数之后，Servlet 的 init() 之前**执行
- @PreDestroy：一般用于释放 bean 持有的资源，在 Servelt 的 **destroy() 之前**执行



被修饰的方法满足以下条件：

- 非静态，无返回值

- 没有任何参数，除非是拦截器(参数为一个 InvocationContext 对象)



### 3. 过滤器



