## TreeNode

### 1. 红黑树

- 根节点为黑色
- 叶节点为黑色
- 不能有两个红节点相连
- 任意一节点到叶节点的黑高度相同



### 2. 实现

#### 2.1 节点

``` java
    // 根据 hash 进行排序
	static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        // 其实维护双向的链表节点
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        
        final TreeNode<K,V> root() {
            for (TreeNode<K,V> r = this, p;;) {
                if ((p = r.parent) == null)
                    return r;
                r = p;
            }
        }
        
        // 通过循环查找对应的key，kc为key的比较类
        final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
        	...
        }
        // 查找节点
        final TreeNode<K,V> getTreeNode(int h, Object k) {
            return ((parent != null) ? root() : this).find(h, k, null);
        }
    }
```

