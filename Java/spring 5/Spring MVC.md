[toc]

## Spring 整合 MVC

### 1. MVC 的作用

- 提供 Controller 调用 Service
- 请求响应的处理
- 接收请求参数
- 控制程序的运行流程
- 视图解析

### 2. 整合 MVC 框架

#### 2.1 整合思路

为保证工厂唯一，应使用如下方法；ServletContext 使用 ServletContextListener 保存共有对象

``` java
ApplicationContext ctx = new ClassPathXmlApplicationContext("xxx.xml");
ServletContext.setAttribute("xxx", ctx); // 保证 ctx 唯一，并共有
```

Spring 使用 ContextLoaderListener 对上述进行了封装

``` java
public class ContextLoaderListener extends ContextLoader implements 		    ServletContextListener {
	... 
}
```

``` xml
<!-- ContextLoaderListener 的使用 -->
<!-- 在 web.xml 中配置-->
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!-- 设置配置文件的位置 -->
<context-param>
    <param-name>contextCongfigLocation</param-name>
    <param-value>classpath:...</param-value>
</context-param>
```

