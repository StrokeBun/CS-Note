[toc]

## Java SE

### 1. JDK、JRE、JVM、JMM

jdk : java 开发工具包，包含了 jre，以及编译器 javac 和调试分析工具

jre : java 运行环境，包含了 jvm 和类库

jvm :  java 虚拟机

jmm ：java 内存模型，规定了线程与内存的关系，是jvm的一个规范

关系：jdk > jre > jvm



### 2. == 和 equals

**== 解析**

对于基本类型，== 直接比较值；对于引用类型，== 比较引用(内存地址，因为 java 只有值传递，实际上 == 就是比较值)

Integer、Long 等进行了缓存，数字范围在 -128 - 127 的包装器会引用同一个对象，== 的结果为 true

**equals解读**

equals 的默认实现即为 == (Object类)，但一般进行 override

<font color=blue>重写 equals 的要求：</font>

1.自反性，x.equals(x) 为 true

2.对称性，x.equals(y) 与 y.equals(x) 结果相同

3.传递性， x.equals(y)、 y.equals(z) 为 true，x.equals(z) 为 true

4.一致性，多次调用的结果相同

5.对于非 null 的 x ，x.equals(null) 为 false

<font color=red>6.重写 hashCode()</font>



### 3.重写 equals() 后为什么要重写 hashCode()?

如果为重写 hashCode()，将违反相等对象拥有相同哈希码的约定条款，在hashMap等容器中将会

存放两个内容相同的对象



### 4. 基本类型与包装类型

**1. 8种基本类型**(java不存在无符号类型)：

- boolean: <font color=red>理论上</font> 1 位(具体见 JVM 实现)，true 或者 false; 不能使用 C++ 与整数判断的方法

- byte: 8 位，范围 -128 - 127

- char: 16 位，存储Unicode码

- short: 16 位整数，范围 -32768 - 32767

- int : 32 位整数

- long: 64 位整数

- float: 32 位浮点数

- double: 64 位浮点数，一般选择 double 而不是 float ，double 的运算速度并不比 float 慢

**2. 包装类型**

包装类型均为 final 类，Byte、Short、Integer、Long对 -128 - 127进行了缓存 (使用 = 赋值或者 valueOf 将会调用缓存)

包装的类的 hashCode()：不大于 32 位的，直接取数值;  64 位的hashCode()将高 32 与低 32 进行异或,

减少哈希冲突

**3. 自动装箱与拆箱**

jdk 1.5 引入的语法糖，实现基本类型与包装类型的转换，但需注意频繁装箱与拆箱带来的性能损失



### 5. final关键字

**<font size=4px>作用</font>**

**1. 修饰类**

该类无法被继承，例如 String 类；final 类的所有方法会被隐式指定为 final 方法

**2.修饰方法**

该方法无法被重写，因不需要动态绑定比非 final 方法快，private 方法隐式为 final 方法

**3. 修饰变量**

必须显式初始化; 修饰基本类型即为常量，修饰引用类型则不能再指向其他对象(对象内容可以修改，与C++ 的 const 有区别), 编译器级别的限制

**作用总结**

(1) 作为常量；(2) <font color=blue>保证线程安全</font>; (3)提高性能



**<font size=4px color=blue>底层原理</font>**

TODO。。。



### 6. final、finally、finalize

三者基本无关系

**1. final**

**2. finally**

使用在异常捕获机制中，finally中的语句块总是会被执行(除了已经 exit, 即虚拟机关闭)，用来释放资源

jdk 1.7 后使用 <font color=blue>AutoCloseable</font> 接口，更加优雅

**3. fInalize**

finalize() 是 Object 类中的方法，是 GC 的一部分，对象被 JVM 回收前会先调用 finalize() 方法

finalize() 只会调用一次，且具有不确定性，不能保证代码执行完，一般不推荐使用



### 7. String、 StringBuffer、StringBuilder、StringJoiner

**1. String**

不可变( final 类)，每次会产生一个新的 String ，优点：不同的字符串变量可以引用字符串池中相同的字符串

```java
String t1 = "hello";
String t2 = "hello";
String t3 = new String("hello");
System.out.println(t1 == t2); // true
System.out.println(t1 == t3); // false，t3在堆上申请
```

jdk 1.8 底层实现为字符数组

```java
private final char value[];
```

jdk 1.9 改为 byte 数组实现，并标识编码方式

```java
/** The value is used for character storage. */
private final byte[] value;

/** The identifier of the encoding used to encode the bytes in {@code value}. */
private final byte coder;
```

**2. StringBuilder**

jdk 1.5 引入，使用了类似 ArrayList 的动态机制( 2 倍扩容+2)，默认容量为 16

通过调用 append() 方法在字符数组后面添加字符串，并返回 <font color=blue>this</font>,

最后通过调用 toString() 返回字符串，速度更快，但线程不安全

**3. StringBuffer**

char[] toStringCache 作为缓存，保存最后依次 toString() 的内存，如果 StringBuffer 进行了修改则为null; 调用 toString() 会先检查 toStringCache 是否为空

StringBuffer 的方法与 StringBuilder 基本相同，但通过 <font color=blue>synchronized</font> 添加锁来保证**线程安全**

**4. StringJoiner**

jdk 1.8 引入的工具类，底层调用了 StringBuilder，可以添加前缀、后缀构建字符串



### 8. 访问修饰符

|           | 当前类 | 同包 | 子类 | 其他 |
| :-------: | :----: | :--: | :--: | :--: |
|  public   |  Yes   | Yes  | Yes  | Yes  |
| protected |  Yes   | Yes  | Yes  |  No  |
|  default  |  Yes   | Yes  |  No  |  No  |
|  private  |  Yes   |  No  |  No  |  No  |



### 9. Java的参数传递机制

java 只存在<font color=blue>值传递</font>，不管是基本类型还是引用类型，引用类型传递是引用，可以通过该引用修改原对象(类似 C++ 的指针)，但修改参数的指向对原引用无影响



## OOP

### 1.三大特性

封装、继承、多态

###  2.重写与重载

重写即 override，子类对父类的方法进行覆盖，必须具有相同的方法名、参数列表、返回类型

重写存在以下 3 点限制：

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。
- 子类方法抛出的异常类型必须是父类抛出异常类型或为其子类型

重载是多个方法名相同但参数不同的情况(返回类型不同不算重载)

### 3. 继承

java 只支持单继承，故引入了接口 interface 实现了多继承的效果

### 4. Java 泛型

Java 的泛型是伪泛型，在编译期间，将进行类型擦除，字节码中不包含类型信息(在字节码中都将使用Object 进行替换)

具体可见 https://www.cnblogs.com/wuqinglong/p/9456193.html

### 5. super 与 this

this 即引用类当前实例，非必要，但可以增加可读性(构造函数)

super 用于访问父类的变量与方法， super() 调用父类的构造方法, 但**必须位于第一行**

this 与 super 均不能用于 static 方法中，this、super属于实例， static 属于类

### 6. 接口与抽象类

区别：

1.接口方法默认是 public, 不允许有实现( jdk 1.8 引入 default 实现)；抽象类中可以有非 abstract 方法

2.接口变量只能为 static 、final，抽象类不一定

3.一个类可以实现多个接口，但只能继承一个抽象类

4.接口方法的默认修饰符为public ( jdk 1.9之后允许定义私有方法)， 抽象方法不允许为 private

