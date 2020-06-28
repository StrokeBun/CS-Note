[toc]

### 1. ArrayList

1.ArrayList底层使用数组实现，提供O(1)的随机访问，**transient** Object[] **elementData**

2.初始化时容量为0，插入第一个元素后容量为<font color=blue>10</font>，自动扩容机制采用<font color=blue>1.5倍</font>扩容

<font color=blue>为什么不采用固定扩容?</font>

如果以固定容量p扩容，假设最坏情况，在初始容量为0的空向量，连续插入n=mp个元素,

于是，在1,p+1,2p+1,3p+1...次插入时需要扩容,扩容带来的复制元素成本为0, p, 2p, 3p, ... , (m-1)p
$$
总体耗时 = p*(m-1)*m/2 = O(n^2)
$$
每次扩容的分摊成本为O(n)

3.使用clear或者删除元素，其实是进行arraycopy(深拷贝)，并将数组尾部元素设置为null, 使用GC回收

4.ArrayList非线程安全，多线程情况下使用CopyOnWriteArrayList类

5.fail-fast?

TODO。。。



### 2. LinkedList

1.底层使用双向链表实现(jdk 1.8之后，jdk 1.6之前是双向循环链表)

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

2.线程不安全



### 3. Vector

1.基本同ArrayList, 但可以指定固定扩容大小，默认采用2倍扩容

2.采用了synchronized修饰其方法，线程安全；但已经不推荐使用

3.readObject是不带锁的，writeObject带锁，与读写锁类似



### ArrayList与LinkedList的对比

1.在随机访问方面，前者是O(1)，后者为O(n)

2.在非尾部插入后删除时，ArrayList需要进行数组复制，LinkedList需要先进行遍历; 故LinkedList适合在头部插入、删除的场合，ArrayList则是尾部

3.均实现了List接口，都是线程不安全的



