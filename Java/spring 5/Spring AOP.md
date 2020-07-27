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

### 3. 切入点

#### 3.1 切入点表达式

##### 3.1.1 方法切入点

``` xml
* *(..) --> 对应所有方法 
* --> 修饰符和返回值
* --> 方法名
() --> 参数表
.. --> 对于参数没有要求


<!-- 定义login方法是切入点-->
<aop:pointcut id="pc" expression="execution(* login(..))"/>
<!-- 非java.lang包中的类型，必须要写全限定名-->
<aop:pointcut id="pc" expression="execution(* register(com.stroke.demo.proxy.User))"/>
```

##### 3.1.2 类切入点

``` xml
<aop:pointcut id="pc" expression="execution(* com.stroke.demo.proxy.
                                  UserServiceImpl.*(..))"/>
```

##### 3.1.3 包切入点

``` xml
<aop:pointcut id="pc" expression="execution(* com.stroke.demo.proxy.
                                  *.*(..))"/>
```

#### 3.2 切入点函数

切入点函数用于执行切入点表达式

- execution：可以执行方法、类、包切入点表达式，但书写麻烦

- args：用于函数参数的匹配

  ``` xml
  args(String, String) = execution(* *(String, String)) 
  ```

- within：主要用于进行类、包切入点表达式匹配

  ``` xml
  within(*..UserServiceImpl) = execution(* *..UserServiceImpl.*(..))
  ```

- @annotation：用于带有注解的函数

  ``` xml
  @annotation(该注解的全限定名) 
  ```

  

### 4. AOP 编程

#### 4.1 概念

AOP：Aspect Oriented Programing，面向切面编程

以切面为基本单位的程序开发，通过切面间的彼此协同，相互调用，完成程序的构建，本质就是动态代理开发

``` markdown
切面 = 切入点 + 额外功能
```

#### 4.2 底层实现

aop 底层实现与动态代理的底层实现相似

##### 4.2.1 核心问题

- 如何创建动态代理类
- Spring 工厂如何加工创建代理对象

##### 4.2.2  JDK 的动态代理

**参数详解**

``` java
/*
	作用: 通过动态字节码技术创建代理类
	参数:
		ClassLoader:类加载器，加载字节码运行，或者创建Class对象，进而创建实例;
					在本方法中由于使用动态字节码技术，故需要借用一个类加载器
		InvocationHandler: 额外功能
		Class<?>[] interfaces: 代理类和原始类实现的共同的接口
*/
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
```

``` java
/*
	作用:实现额外功能
	返回值:原始方法的返回值
	参数：
		Method: 原始方法
		Object[]:原始方法的参数
*/
public interface InvocationHandler {
    Object invoke(Object proxy, Method var2, Object[] var3) throws Throwable;
}

// 实现类似 MethodInterceptor
@Override
public Object invoke(Object proxy, Method var2, Object[] var3) throws Throwable {
    // 额外功能。。。
    Object ret = method.invoke(原始类, args);
    // 额外功能。。
    return ret;
}
```

**编码实现**

``` java
        // 1.创建原始对象
        UserService userService = new UserServiceImpl();

        // 2. 实现额外功能
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object o, Method method, Object[] objects) throws Throwable {
                Object ret = method.invoke(userService, objects);
                System.out.println("-------JDK Proxy log--------");
                return ret;
            }
        };
        // 3. JDK创建代理类
        UserService proxy = (UserService)            Proxy.newProxyInstance(JDKProxyTest.class.getClassLoader(),
                userService.getClass().getInterfaces(), handler);
        // 4.调用
        proxy.login("bzzb", "bzzb");
```

##### 4.2.3 CGlib 的动态代理