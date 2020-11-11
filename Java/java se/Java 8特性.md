## Java 8特性

### 1. Lambda表达式

#### 1.1 行为参数化

行为参数化的三个操作：

- 策略模式：需要创建多个策略类
- 匿名类：过于笨重
- lambda 表达式

``` java
// 使用策略模式
// 苹果重量策略
public class AppleWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeigth() > 150;
    }
}

// 苹果颜色策略
public class AppleColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
}

// 苹果过滤器
public static List<Apple> filterApples(List<Apple> inventory,
                                       ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

``` java
// 匿名类实现
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
   public boolean test(Apple apple) {
       return "red".equals(apple.getColor());
   } 
});
```

``` java
// lambda 实现
List<Apple> result = filterApples(inventory, (Apple apple) -> 
                                  "red".equals(apple.getColor()));
```

对上述操作使用泛型，即可得到泛用的过滤函数

``` java
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e: list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}

public interface Predicate<T> {
    boolean test(T e);
}
```

#### 1.2 函数式接口

lambda 表达式主要用于函数式接口，该接口只定义一个抽象方法

JDK 1.8 引入 <a color="blue">`@FunctionalInterface`</a> 注解用来标识函数式接口，在编译期进行检查

#### 1.3 限制

lambda 表达式允许使用 **自由变量**，即在作用域外的变量，闭包特性。对于引用和静态变量没有限制，而**对于局部变量必须为 final 或者事实上是 final**

``` java
// 可以通过编译
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);

// 无法通过编译
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```

原因：因为引用是存储在堆，而局部变量保存在线程栈上。lambda 是在一个线程 t1 中使用，如果 lambda 访问局部变量没有限制，那么该局部变量被分配该变量的线程 t2 回收后，t1 可能会访问该变量，导致错误。

#### 1.4 方法引用

方法引用允许使用已有的方法创建 lambda，显式指明方法名称，提升可读性。

``` java
// 使用labda
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 使用方法引用和java.util.Comparator.comparing
inventory.sort(comparing(Apple::getWeight));
```

``` java
// 指向Apple()构造函数
Supplier<Apple> c1 = Apple::new;
// 创建对象
Apple a1 = c1.get();
```

