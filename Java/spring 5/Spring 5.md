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
UserService userService = (UserService)clazz.newInstance();
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

