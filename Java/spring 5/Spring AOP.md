[toc]

## Spring AOP 编程

### 1. 静态代理

#### 1.1 为什么需要代理？

除了 DAO、业务等功能之外，还有日志等额外功能；

- 对于 Controller 来说，额外功能应该在 Service 层实现，调用简单
- 对于软件设计者来说，Service 层应该不包含额外功能，维护方便

故引入一个代理

- 实现额外功能
- 调用原始类的核心功能

#### 1.2 静态代理设计模式

##### 1.2.1 名词解释

- 目标类、原始类：实现核心功能的业务类
- 目标方法、原始方法：即核心功能
- 额外功能：日志、事务等

##### 1.2.2 核心要素

代理类和目标类**<font color=blue>实现共同的接口</font>**

例如 Thread 就是代理类，实现了 Runnable 接口，核心功能即 run() 方法，额外功能即线程创建等功能

静态代理的特点：**<font color=blue>一个目标类对应一个代理类</font>**

##### 1.2.3 静态代理存在的问题

- 代理类过多，不利于项目管理
- 额外功能的维护性差，如每个代理类都实现了日志，当需要修改日志时，需要修改所有代理类



### 2. Spring动态代理开发

#### 2.1 开发步骤

- 创建原始对象(例如 UserServiceImpl )

- 额外功能，实现 MethodBeforeAdvice 接口

- 实现切入点：额外功能加入的位置

  ``` java
  /*
  	Method:额外功能所增加给的原始方法
  	Object[]: 原始方法的参数数组
  	Object: 原始对象
  */
  public interface MethodBeforeAdvice extends BeforeAdvice {
      void before(Method var1, Object[] var2, @Nullable Object var3) throws Throwable;
  }
  ```

- 组装

  ``` xml
      <aop:config>
          <!--所有的方法，都加入切入点，加入额外功能-->
          <aop:pointcut id="pc" expression="execution(* *(..))"/>
          <!-- 组装：目的把切入点与额外功能进行整合-->
          <aop:advisor advice-ref="before" pointcut-ref="pc" />
      </aop:config>
  ```

- 调用，<font color=blue>通过原始对象的 id 值获得的是代理对象，需要用接口存储</font>

  ``` java
          UserService userService = (UserService) ctx.getBean("userService");
          userService.register(new User());
  ```

#### 2.2 细节分析

1.Spring 创建的动态代理类在哪里？

Spring 通过**动态字节码**技术，在 JVM 创建并运行在 JVM 内部，等程序结束后销毁

动态字节码：通过第三方框架，在 JVM 中创建对应类的字节码，进而创建对象

2.动态代理的优势

- 降低了代理类开发的复杂度
- 增强了额外功能的可维护性

#### 2.3 MethodInterceptor

``` java
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
    /*
		额外功能在invoke中实现
		参数 MethodInvocation: 增加额外功能的原始方法
	    	invocation.proceed() 对应原始方法的运行，可在运行之前或之后实现额外功能
		返回值：原始方法的返回值
	*/
    Object invoke(MethodInvocation var1) throws Throwable;
}

// 实现
@Override
public Object invoke(MethodInvocation var1) throws Throwable {
    // 额外功能。。。
    Object ret = invocation.proceed();
    // 额外功能。。
    return ret;
}
```

**细节分析**

- 如果额外功能运行在抛出异常

  ``` java
          Object ret = null;
          try {
              Object ret = methodInvocation.proceed();
          } catch (Exception e) {
              // 额外功能
          }
          return ret;
  ```

### 3. 切入点详解

