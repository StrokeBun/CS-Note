## CAS

### 1. 介绍

CAS：Compare and Swap，是一种乐观锁实现

核心思路：CAS 有 3 个操作数，内存位置 V，期望值 A，新值 B，如果 V 中的值等于 A，则将 V 中的数据更新为 B；否则则不进行操作，最终返回 V 的数值



### 2. JDK 的 CAS 支持

sun.misc.Unsafe 提供了 CAS 支持

``` java
    /*
    * var1: 要操作的对象
    * var2: 要操作的属性的地址偏移量
    * var1 + var2 可以确定内存位置V的值
    * var4: 期望值A
    * var5: 新值B
    */
    public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

    public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```



在 intel x86 平台下，Java 的 CAS 通过 cmpxchg 实现

``` markdown
1. 不同核心数量的实现
	- 多处理器: 添加lock前缀
	- 单处理器: 不添加lock前缀
2. lock前缀说明（结合MSEI协议）: 
	- 如果待访问的内存区域已经位于cache中，并且Cache Line的状态为已修改或独占，则直接更新(读写时  	   Cache Line会被锁定，其他CPU无法更新数据); 否则锁住总线进行更新
	- 禁止该指令前后的读写指令重排序
	- 将写缓冲区的数据刷新到内存
```



### 3. CAS 的不足

- 循环时间长，高并发下开销大

- 只能保证一个变量的原子操作

- ABA 问题：一个变量的数据由 A 改为 B 再改为 A，CAS 无法分辨

  解决方案：增加了版本号 version

  JDK 提供了 AtomicStampedReference 作为解决方案

  ``` java
  public class AtomicStampedReference<V> {
  
      // 内部类Pair包括引用和版本号
      private static class Pair<T> {
          final T reference;
          final int stamp;
          ...
      }
  
     private volatile Pair<V> pair;
     
     /*
     * expectedReference: 期望引用
     * newReference: 新值引用
     * expectedStamp: 期望版本号
     * newStamp: 新版本号
     */
     public boolean compareAndSet(V   expectedReference,
                                   V   newReference,
                                   int expectedStamp,
                                   int newStamp) {
          Pair<V> current = pair;
          return
              expectedReference == current.reference && // 引用一致
              expectedStamp == current.stamp && // 版本一致
              ((newReference == current.reference &&
                newStamp == current.stamp) ||
               casPair(current, Pair.of(newReference, newStamp))); // 设置新值
      }
      
      private boolean casPair(Pair<V> cmp, Pair<V> val) {
          return UNSAFE.compareAndSwapObject(this, pairOffset, cmp, val);
      }
  }
  ```

  