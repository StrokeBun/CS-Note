[toc]

## synchronized

### 1. 介绍

synchronized 是 java 中的可重入悲观锁，在 jdk 1.6引入了偏向锁、轻量锁进行锁升级降低了性能消耗

<img src="img/java monitor.jpg" style="zoom:80%" />

synchronized 的三种加锁方式

- 普通方法，相当于给**实例对象**加锁，进入同步代码之前要获得当前实例的锁
- 静态方法，相当于给**类对象**加锁
- 代码块，需要指定加锁对象，对**给定对象**加锁



### 2. 原理

#### 2.1 JVM 层次

java 中的锁，其实就是一个 monitor 对象，java 的每个对象都携带 monitor，monitor 存放在**对象头**的 

Mark Word  中，下图是 Mark Word 中与锁相关的部分

<img src="img/mark word中锁相关部分.jpg" />

``` c++
 ObjectMonitor() {
    _header       = NULL;
    _count        = 0; // 该线程获取锁的次数
    _waiters      = 0,
    _recursions   = 0; // 锁重入次数
    _object       = NULL;
    _owner        = NULL; // 持有ObjectMonitor对象的线程
    _WaitSet      = NULL; // wait状态线程列表
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; // 等待锁释放的线程队列
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

在进入同步代码块前，会使用 monitorenter 指令尝试获取锁，获取失则线程挂起，进入 WaitSet

结束同步代码块后，执行 monitorexit  释放锁，如果调用 wait 则进入 WaitSet

<img src ="img/moniterenter.jpg" />

<img src ="img/moniterexit.jpg" />

#### 2.2 CPU 级别

JVM 底层使用 Atomic::cmpxchg_ptr 尝试把 monitor 的 _owner 字段设置为当前线程，

依赖了 CPU 底层的 <a href="../jdk源码/util/CAS.md">CAS 原语</a>，使用 lock 指令实现



### 3. 锁升级

锁升级的流程

<img src="img/锁升级过程.jpg"/>

#### 3.1 偏向锁

偏向锁的作用是当线程访问同步代码时，只需要判断 Mark Word 中偏向锁线程 ID 是否为自身，是则直接进入同步代码块

偏向锁记录流程

- 线程抢到同步锁
- 对象 Mark Word 设置偏向标志为为 1
- 对象 Mark Word 记录线程 ID
- 进入偏向状态

当有线程竞争偏向锁，并且发现线程 ID 不是自身的话，将尝试获取锁

- 获取成功，更改对象 Mark Word 的线程 ID，保持偏向状态
- 获取失败，升级为轻量级锁

#### 3.2 轻量级锁

轻量级锁通过自旋实现，在 jdk 1.7 后默认开启

自旋次数可以通过 JVM 设置，默认为 10 次，当自旋失败后，将会升级为重量级锁

#### 3.3 重量级锁

重量级锁开始使线程挂起阻塞，减少 CPU 消耗，需要进入内核态

