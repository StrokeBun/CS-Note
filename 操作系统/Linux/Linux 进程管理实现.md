## Linux 进程管理实现

### 1. 程序的二进制格式

代码到进程的演变过程

<img src="img/代码到进程的演变过程.jpg" style="zoom:60%" />

Linux 的二进制程序格式为 **ELF**，可分为

- 可重定位文件：中间文件，链接之后形成可执行文件

  <img src="img/可重定位文件ELF格式.jpg" style="zoom:80%" />

  文件格式

  - .text：放编译好的二进制可执行代码
  - .data：已经初始化好的全局变量
  - .rodata：只读数据，例如字符串常量、const 的变量
  - .bss：未初始化全局变量，运行时会置 0
  - .symtab：符号表，记录的则是函数和变量
  - .strtab：字符串表、字符串常量和变量名

- 可执行文件：可执行文件中的各个段由多个可重定位文件链接而来，载入内存后即可运行

  <img src="img/可执行文件ELF格式.jpg" style="zoom:80%" />

- 共享对象文件/动态链接库：静态库代码段被多个程序使用将在内存中产生多个副本，动态链接库通过 **在内存中保存库的引用** 解决该问题



### 2. 进程与线程概述

#### 2.1 进程

**进程状态**：

- TASK_RUNNING：进程在 CPU 上执行或准备执行
- TASK_INTERRUPTIBLE：可中断的等待状态，当等待的资源被释放或者收到一个信号，可唤醒该进程
- TASK_UNINTERRUPTIBLE：不可中断的等待状态，只能等待资源释放，不可被信号唤醒，包括KILL 信号
- TASK_KILLABLE：可以终止的新等待状态，类似 TASK_UNINTERRUPTIBLE，但可被 KILL 唤醒
- TASK_STOPPED：接收到 SIGSTOP、SIGTSTP、SIGTTIN、SIGTTOU 信号进入该状态
- TASK_TRACED：被 debugger 等进程监视会进入该状态

用户态进程的顶级父进程为 **systemd** 进程，内核态进程的顶级父进程为 **kthreadd** 进程

<img src="img/进程树.jpg" style="zoom:80%" />

#### 2.2 线程

<img src="img/线程的创建与运行过程.jpg" style="zoom:75%" />

线程数据类型：

- 本地数据：存放在线程独立的栈中，例如局部变量
- 全局数据：线程共享，例如进程中的全局变量
- 线程私有数据：采用一键多值的形式，各个线程通过相同的 key 得到不同的 value



### 3. 进程数据结构

Linux 进程与线程都使用 **task_struct** 统一维护

``` c
struct tast_struct {
    /* id相关部分，如果只有一个主线程，则pid = tgid, group_leader为该主线程 */
    pid_t pid; // 进程id
    pid_t tgid; // 线程组id
    struct task_struct *group_leader;
    /* 任务状态 */
    volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
    int exit_state;
    unsigned int flags;
    /* 调度相关 */
    int	on_rq; // 是否在运行队列上
    // 优先级
    int prio;
    int	static_prio;
    int	normal_prio;
    unsigned int rt_priority;
    // 调度实体
    struct sched_entity	se;
    struct sched_rt_entity rt;
    struct sched_dl_entity dl;
    unsigned int policy; // 调度策略
    /* 亲缘关系 */
    struct task_struct __rcu *real_parent; /* real parent process */
    struct task_struct __rcu *parent; // 当前父进程，通过与real_parent相同
                                      // 终止时须向父进程发送信号
    struct list_head children; // 子进程链表头部
    struct list_head sibling;  // 兄弟进程链表的下一个元素
    /* 用户态与内核态相关 */
    struct thread_info	thread_info; // 保存了task_struct的节点，
                                     // 如果这个进程正在运行，可用于查找该进程的相关信息
    void  *stack; // 内核栈
    ... 
}
```

``` c
struct thread_info {
	struct task_struct	*task;		/* main task structure */
	__u32			flags;		/* low level flags */
	__u32			status;		/* thread synchronous flags */
	__u32			cpu;		/* current CPU */
	mm_segment_t		addr_limit;
	unsigned int		sig_on_uaccess_error:1;
	unsigned int		uaccess_err:1;	/* uaccess failed */
};
```

内核栈结构如下，pt_regs 即为进入内核态时保存的寄存器信息

<img src="img/内核栈结构.jpg" style="zoom:60%"/>

<img src="img/进程数据结构.jpg" style="zoom:90%"/>



### 4. 进程调度

