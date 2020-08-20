## ThreadLocal

### 1. 介绍

ThreadLocal 主要作用是用于数据隔离，使该数据只属于当前线程，在多线程环境中，可以防止自己的变量被修改，每个 Thread 中保存了一个 ThreadLocal.ThreadLocalMap 用于储存该线程的多个 ThreadLocal 对象

``` java
    // Thread.Class
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
```

Spring 的事务隔离采用了 ThreadLocal，保证单个线程中的数据库操作使用的同一个数据库连接

### 2. 属性

``` java
    // 当前ThreadLocal所对应的HashCode
    private final int threadLocalHashCode = nextHashCode();
    // 通过AtomicInteger生成HashCode
    private static AtomicInteger nextHashCode =
        new AtomicInteger();
    /*
     * 当前线程新增一个ThreadLocal，其hashCode会递增一个固定值
     * ThreadLocalMap 通过开放寻址解决哈希冲突
     * 该值在长度为 2^n 的数组中能尽可能降低哈希冲突
     */
    private static final int HASH_INCREMENT = 0x61c88647;
```

### 3. ThreadLocalMap

``` java
    static class ThreadLocalMap {
        // Entry 是个弱引用，用来优化回收对象
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;
            // key为ThreadLocal
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
       	// 初始化容量为 16
        private static final int INITIAL_CAPACITY = 16;
        // 负载因子默认为 2/3
        private int threshold; // Default to 0
        private void setThreshold(int len) {
            threshold = len * 2 / 3;
        }
        /* 其他常量与HashMap相似，故省略*/
    }
```

``` java
	/**
	 * ThreadLocal.ThreadLocalMap.getEntry
	 */
	private Entry getEntry(ThreadLocal<?> key) {
        	// 数组固定为 2^n
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            return e;
        else
            return getEntryAfterMiss(key, i, e); // 通过开放寻址获得Entry
     }

        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    /*
                    * 从索引staleSlot开始，遍历一段【连续】的元素，清理其中的垃圾值
                    * 如果存在哈希冲突，并且前一个元素是垃圾值，则往前移动，覆盖冲突
                    */
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len); // 获得下一个Entry
                e = tab[i];
            }
            return null;
        }
```

``` java
        /** 
         * ThreadLocal.ThreadLocalMap.set
         */
		private void set(ThreadLocal<?> key, Object value) {

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            // 通过开放寻址解决哈希冲突
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();
				// 覆盖value
                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    // 向后遍历，给ThreadLocal安排位置并清除垃圾
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash(); // 先清除垃圾，如果仍大于阈值，则扩容
        }
```

### 4. get 和 set

``` java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

``` java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        // ThreadLocalMap仍未初始化，则创建一个，并插入值
        else
            createMap(t, value);
    }
```

### 5. 内存分布

ThreadLocal 存放在每个线程的栈内存上，栈内存线程私有

因为线程实例存放在 JVM 的堆，故实际 ThreadLocal 的实例仍存放在堆上，但设置成了该线程可见

如果想共享线程的ThreadLocal ，可使用 InheritableThreadLocal

### 6. 内存泄漏问题

ThreadLocalMap 的生命周期与创建该 ThreadLocal 的线程一样长，如果没有手动删除，则 value 将一直无法被回收，导致内存泄漏

例如线程池中的线程是复用的，运行结束后仍然存活，ThreadLocal 设定的 value 也一直存活

解决方案：

``` java
ThreadLocal<String> localName = new ThreadLocal();
try {
    localName.set("张三");
    ……
} finally {
    // 在finally里中调用remove删除
    localName.remove();
}
```

**为什么 ThreadLocal 设置为弱引用？**

弱引用会被 GC 自动回收，ThreadLocalMap 的一系列操作都有清除垃圾的环节，会设置已经被回收的 slot 为 null，可以减少手动回收，帮助优化无用对象的回收