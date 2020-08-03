[toc]



## Redis 底层实现

### 1. 字符串

```c
struct sdshdr {
    // buf数组已使用的字节数
    int len;
    // buf数组未使用的字节数
    int free;
    char buf[];
}
```

sdshdr 相比 C 字符串的优势：

- 计数方式不同，C 字符串获取长度时间复杂度 O(n)，sdshdr 为 O(1)

- 杜绝了缓冲区溢出

- 加入了**空间预分配**和**惰性空间释放**

  - 空间预分配：对 sdshdr 拓展时，会分配多余的 free空间 + 1 byte，其余 1 byte 用于存空字符
    - 如果 len 小于 1MB，则 free = len
    - 如果大于等于 1MB， 则 free = 1MB
  - 惰性空间释放：sdshdr 缩减后，不会立即回收多余空间

- 二进制安全，对于二进制文件中存在 '\0' , C 字符串会截断，sdshdr 则不存在这个问题

### 2. 链表

``` c
typedef struct listNode {
    struct listNode * pre;
    struct listNode * next;
    // 采用void* 可以保存任意类型
    void * value;
}
```

``` c
typedef struct list {
    listNode * head;
    listNode * tail;
    // 节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void *(*free)(void *ptr);
    // 节点值对比函数
    void *(*match)(void *ptr, void *key);
}
```

### 3. 字典

#### 3.1 字典实现

redis 的字典采用哈希表实现，哈希表中包含多个哈希表节点，一个哈希表节点即为一个 key-value 对

##### 3.1.1 哈希表

``` c
typedef struct dictht {
    // 哈希表数组
    dictEntry ** tabble;
    // 哈希表大小
    unsigned long size;
    // 哈希表掩码，总等于 size-1
    unsigned long sizemask;
    // 已有节点数量
    unsigned long used;
} dictht;
```

##### 3.1.2 哈希表节点

``` c
typedef struct dictEntry {
    void * key;
    // value可以为指针、无符号64位整数、有符号64位整数
    union {
        void * val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 采用拉链法解决冲突
    struct dictEntry *next;
}
```

##### 3.1.3 字典

``` c
typedef struct dict {
    // 类型特定函数，保存了操作该类型的函数
    dictType *type;
    // 保存了传给该类型的可选参数
    void *pridata;
    // ht[0]存储数据，ht[1]用于rehash复制
    dictht ht[2];
    // 是否在rehash, 如果正在rehash，则值为-1
    int rehashidx;
}
```

``` c
typedef struct dictType {
    // 哈希函数
    unsigned int (*hashFunction)(const void *key);
    // 复制key的函数
    void *(*keyDup)(void *privdata, const void *key);
    // 复制value的函数
    void *(*valDup)(void *privdata, const void *obj);
    // 对比key的函数
    int (*keyCompare)(void *privdata, const void *key1, const void* key2);
    // 销毁key的函数
    void (*keyDestructor)(void *privdata, void *key);
    // 销毁value的函数
    void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

#### 3.2 哈希算法

``` c
// 哈希值计算过程
hash = dict->type->hashFunction(key);
// 计算数组的下标, x为 0/1
index = hash & dict->ht[x].sizemask;
```

当用作 redis 的底层实现时，采用的 hashFunction 是 MurmurHash2 算法

#### 3.3 rehash

redis rehash 的步骤：

- 为 ht[1] 分配空间
  - 如果执行是**拓展**操作，则 ht[1] 的大小为大于等于 **ht[0].used * 2** 的最小的 **2 的 n 次方**
  - 如果执行是**收缩**操作，则 ht[1] 的大小为大于等于 **ht[0].used** 的最小的 **2 的 n 次方**

- 将 ht[0] 的所有键值对 rehash 到ht[1] 上
- 释放 ht[1]，将 ht[1] 设置为 ht[0]，ht[1] 创建一个空表，为下次 rehash 做准备



拓展时机：

- 目前没有执行 BGSAVE 或 BGREWRITEAOF，且负载因子大于等于 1
- 目前正在执行 BGSAVE 或 BGREWRITEAOF，且负载因子大于等于 5

解释：

对于执行下述命令时，redis创建了子进程，操作系统大多采用 copy-on-write 来优化子进程的使用效率，redis 提高了负载因子的上限，避免子进程存在期间哈希表进行拓展，减少不必要的内存写入

copy-on-write:

复制一个对象时，不是真正进行复制，而是在新对象的内存映射表(Translation Table)中指向同原对象相同的位置，copy-on-write位设置为1，在对这个对象执行读操作的时候，内存数据没有变动，直接执行就可以了。执行写操作的时候，才将原对象复制一份到新地址，并且同时修改新对象的内存映射表到这个新位置，然后在进行写入操作

收缩时机：负载因子小于 0.1

