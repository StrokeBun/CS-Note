## Object

```java
// Java所有类的父类
public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    // 获得当前对象所属类的类对象
    public final native Class<?> getClass();
    
    // 获得哈希码，默认由地址生成，在hash容器中广泛使用
    // 原则1: 相等的对象，hashCode返回值必须相同
    // 原则2: 重写equal后必须重写hashCode
    public native int hashCode();
    
    // 判断是否相等，默认比较两个对象的引用
    public boolean equals(Object obj) {
        return (this == obj);
    }

    /** 
     * 浅拷贝，使用时往往需要重写为public形式。
     * 注：要求被克隆的对象类实现Cloneable接口 
     */
    protected native Object clone() throws CloneNotSupportedException;

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    
    // 随机唤醒某个具有相同锁的对象进入争锁状态
    public final native void notify();
	
    // 唤醒所有具有相同锁的对象进入争锁状态
    public final native void notifyAll();

    /*
     * wait使得调用该方法的线程放弃锁，进入WAITING或TIMED_WAITING状态
     *
     * wait方法应当配合synchronized一起使用：
     *
     * 示例一：
     * synchronized void fun(){
     *   try {
     *      wait(1000);
     *   } catch(InterruptedException e) {
     *      e.printStackTrace();
     *   }
     * }
     *
     * 示例二：
     * synchronized(object) {
     *   try {
     *     object.wait(1000);
     *   } catch(InterruptedException e) {
     *     e.printStackTrace();
     *   }
     * }
     *
     * wait让当前线程陷入等待的同时，释放其持有的锁，以便让其他线程争夺锁的控制权
     *
     * wait线程醒来的条件：
     * 1. 超时
     * 2. 被notify()或notifyAll()唤醒
     * 3. 在其他线程中调用该线程的interrupt()方法
     *
     * 注：
     * wait方法持有的锁是当前wait所处的上下文的对象（某个栈帧中的对象）
     * 如果wait持有的锁与当前上下文中的锁不一致，或者wait和notify用的锁不一致，会触抛出
     * InterruptedException异常
     */
    public final native void wait(long timeout) throws InterruptedException;

    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }

  
    public final void wait() throws InterruptedException {
        wait(0);
    }

    // 对象在被GC回收后执行的清理操作，只会调用一次，不推荐使用
    protected void finalize() throws Throwable { } 
}
```