[toc]

## Java SE

### 1. JDK、JRE、JVM、JMM

jdk: java开发工具包，包含了jre，以及编译器javac和调试分析工具

jre: java运行环境，包含了jvm和类库

jvm: java虚拟机

jmm：java内存模型，规定了线程与内存的关系，是jvm的一个规范

关系：jdk>jre>jvm



### 2. == 和 equals

**==解析**

对于基本类型，==直接比较值；对于引用类型，==比较引用(地址)

Integer、Long等进行了缓存，数字范围在-128-127的包装器会引用同一个对象，==的结果为true

**equals解读**

equals的默认实现即为==(Object类)，但一般进行override

<font color=blue>重写equals的要求：</font>

1.自反性，x.equals(x)为true

2.对称性，x.equals(y)与y.equals(x)结果相同

3.传递性， x.equals(y)、 y.equals(z)为true，x.equals(z)为true

4.一致性，多次调用的结果相同

5.对于非null的x，x.equals(null)为false

<font color=red>6.重写hashCode()</font>



### 3.重写equals()后为什么要重写hashCode()?

如果为重写hashCode()，将违反相等对象拥有相同哈希码的约定条款，在hashMap等容器中将会

存放两个内容相同的对象



### 4. 基本类型与包装类型

**1. 8种基本类型**(java不存在无符号类型)：

boolean: 1位，true或者false; 不能使用C++与整数判断的方法

byte: 8位，范围 -128 - 127

char: 16位，存储Unicode码

short: 16位整数，范围-32768 - 32767

int : 32位整数

long: 64位整数

float: 32位浮点数

double: 64位浮点数，一般选择double而不是float，double的运算速度并不比float慢

**2. 包装类型**

包装类型均为final类，Byte、Short、Integer、Long对 -128 - 127进行了缓存

包装的类的hashCode()：不大于32位的，直接取数值; 64位的hashCode()将高32与低32进行异或,

减少哈希冲突

**3. 自动装箱与拆箱**

jdk 1.5引入的语法糖，实现基本类型与包装类型的转换，但需注意频繁装箱与拆箱带来的性能损失



### 5. final关键字

**<font size=4px>作用</font>**

**1. 修饰类**

该类无法被继承，例如String类；final类的所有方法会被隐式指定为final方法

**2.修饰方法**

该方法无法被重写，因不需要动态绑定比非final方法快，private方法隐式为final方法

**3. 修饰变量**

必须显式初始化; 修饰基本类型即为常量，修饰引用类型则不能再指向其他对象(对象内容可以修改，与C++的const有区别), 编译器级别的限制

**作用总结**

(1) 作为常量；(2) <font color=blue>保证线程安全</font>; (3)提高性能



**<font size=4px color=blue>底层原理</font>**

TODO。。。



### 6. final、finally、finalize

三者基本无关系

**1. final**

**2. finally**

使用在异常捕获机制中，finally中的语句块总是会被执行(除了已经exit, 即虚拟机关闭)，用来释放资源

jdk 1.7后使用<font color=blue>AutoCloseable</font>接口，更加优雅

**3. fInalize**

finalize()是Object类中的方法，是GC的一部分，对象被JVM回收前会先调用finalize()方法

finalize()只会调用一次，且具有不确定性，不能保证代码执行完，一般不推荐使用



### 7. String、 StringBuffer、StringBuilder、StringJoiner

**1. String**

不可变(final类)，每次会产生一个新的String，优点：不同的字符串变量可以引用字符串池中相同的字符串

```java
String t1 = "hello";
String t2 = "hello";
String t3 = new String("hello");
System.out.println(t1 == t2); // true
System.out.println(t1 == t3); // false，t3在堆上申请
```

底层实现为字符数组(private final char value[])

**2. StringBuilder**

jdk 1.5引入，使用了类似ArrayList的动态机制(2倍扩容+2)，默认容量为16

通过调用append()方法在字符数组后面添加字符串，并返回<font color=blue>this</font>,

最后通过调用toString()返回字符串，速度更快，但线程不安全

**3. StringBuffer**

char[] toStringCache 作为缓存，保存最后依次toString()的内存，如果StringBuffer进行了修改则为null; 调用toString()会先检查toStringCache是否为空

StringBuffer的方法与StringBuilder基本相同，但通过<font color=blue>synchronized</font>添加锁来保证**线程安全**

**4. StringJoiner**

jdk 1.8引入的工具类，底层调用了StringBuilder，可以添加前缀、后缀构建字符串



### 8. 访问修饰符

|           | 当前类 | 同包 | 子类 | 其他 |
| :-------: | :----: | :--: | :--: | :--: |
|  public   |  Yes   | Yes  | Yes  | Yes  |
| protected |  Yes   | Yes  | Yes  |  No  |
|  default  |  Yes   | Yes  |  No  |  No  |
|  private  |  Yes   |  No  |  No  |  No  |



### 9. Java的参数传递机制

java只存在<font color=blue>值传递</font>，不管是基本类型还是引用类型，引用类型传递是引用，可以通过该引用修改原对象(类似C++的指针)，但修改参数的指向对原引用无影响



## OOP

### 1.三大特性

封装、继承、多态

###  2.重写与重载

重写即override，子类对父类的方法进行覆盖，必须具有相同的方法名、参数列表、返回类型

重载是多个方法名相同但参数不同的情况(返回类型不同不算重载)

### 3. Java多继承

java只支持单继承，故引入了接口interface实现了多继承的效果

