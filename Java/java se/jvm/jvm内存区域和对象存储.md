## jvm内存区域和对象存储

### 1. 内存区域

- 运行时内存区
  - 程序计数器：线程私有，类似汇编的 pc 指针
  - java 虚拟机栈：线程私有，存储局部变量、方法调用等，溢出则抛出 `StackOverflowError`
  - 本地方法栈：记录 Native 的方法调用，溢出时将抛出 `StackOverflowError`
  - 堆：对象分配内存的区域，溢出抛出 `OutOfMemoryError`
  - 方法区：存储 class 对象，jdk 1.8 之前还存储常量池，jdk 1.8 以及之后常量池位于元空间（直接内存），溢出抛出 `OutOfMemoryError`

- 直接内存：内含元空间，也用于 NIO 堆外内存，溢出抛出 `OutOfMemoryError`

<img src="img/jvm内存区.png">



### 2. 对象存储



<img src="img/内存区域和对象存储.svg">