# 内核如何统一管理 IPC 资源

> 这一篇是进程间通信的"上帝视角"。看懂它，你就明白 Linux 内核是怎么用 C 语言写出面向对象的效果的。

## 现象：三套接口长得一模一样

```c
int shmget(key_t key, size_t size, int shmflg);   // 共享内存
int msgget(key_t key, int msgflg);                // 消息队列
int semget(key_t key, int nsems, int semflg);     // 信号量
```

`shmctl` / `msgctl` / `semctl` 也是同样的风格。
它们都用 `key` 定位，都返回一个 `int` 类型的 id，都有 `IPC_CREAT` / `IPC_EXCL` / `IPC_RMID`。

**这不是巧合，而是内核有意为之的统一抽象。**

## 三个"子类"结构体

内核里，三种资源各有一个重量级结构体：

| 资源 | 结构体 |
| :--- | :--- |
| 共享内存 | `struct shmid_kernel` |
| 消息队列 | `struct msg_queue` |
| 信号量 | `struct sem_array` |

它们的内容完全不同（共享内存要记页信息，消息队列要记链表，信号量要记计数数组），但**第一个成员完全相同**：

```c
struct kern_ipc_perm {
    spinlock_t  lock;      // 保护本结构体的自旋锁
    int         deleted;   // 是否已被标记删除
    key_t       key;       // 用户层的"暗号"
    uid_t       uid;       // 所有者
    gid_t       gid;       // 所属组
    uid_t       cuid;      // 创建者
    gid_t       cgid;
    mode_t      mode;      // 权限位
    unsigned long seq;     // 序号，防止 id 被复用时的 ABA 问题
};
```

```c
struct shmid_kernel {
    struct kern_ipc_perm shm_perm;   // ← 必须第一个
    struct file         *shm_file;   // 指向 tmpfs 中的特殊文件
    unsigned long        shm_nattch; // 当前挂载数
    // ...
};

struct msg_queue {
    struct kern_ipc_perm q_perm;     // ← 必须第一个
    struct list_head     q_messages;
    // ...
};

struct sem_array {
    struct kern_ipc_perm sem_perm;   // ← 必须第一个
    int                  sem_nsems;  // 信号量个数
    struct sem          *sems;       // 柔性数组，动态扩展
    // ...
};
```

## 为什么必须是第一个成员

C 语言的结构体成员按声明顺序**连续**排列在内存里，**结构体的地址 == 第一个成员的地址**。

```
struct shmid_kernel 的内存布局：
┌─────────────────────────────┐ ← 地址 0x1000（也是 shm_perm 的地址）
│  struct kern_ipc_perm       │
├─────────────────────────────┤ ← 地址 0x1000 + sizeof(kern_ipc_perm)
│  struct file *shm_file      │
│  unsigned long shm_nattch   │
│  ...                        │
└─────────────────────────────┘
```

所以：

- 把 `struct shmid_kernel*` 强转成 `struct kern_ipc_perm*` → 合法，拿到公共头
- 把 `struct kern_ipc_perm*` 强转回 `struct shmid_kernel*` → **也合法**，因为公共头就在对象的最开头，指针值根本没变

这就是 C 语言里实现"继承"的标准手法：**基类放在最前面**。

## 统一数组：`ipc_id_ary`

内核不会为三种资源各建一张表，而是维护一个全局的**柔性数组**（本质是基数树/哈希表的历史实现）：

```c
struct ipc_id_ary {
    int size;
    struct kern_ipc_perm *p[0];   // 柔性数组：零长度占位
};
```

> **柔性数组（Flexible Array）的意义**：`p[0]` 不占结构体空间，但允许在 `kmalloc(sizeof(struct ipc_id_ary) + n * sizeof(void*))` 时，把 `p[0]` 之后的空间当作数组用。一次内存分配就把"头部 + 数组"全搞定，减少碎片。

`shmget` 返回的那个 `shmid`，**就是这张表的下标**。

```
ipc_id_ary.p[]:
  [0] → struct shmid_kernel  (强转成 kern_ipc_perm 后挂在表里)
  [1] → struct msg_queue
  [2] → struct sem_array
  [3] → ...
```

## 一次系统调用的完整流转

以 `semop(semid, ...)` 为例：

```
用户态  semop(semid, &buf, 1)
          ↓ int 0x80 / syscall
内核态  sys_semop()
          ↓ 用 semid 当下标
        perm = ipc_id_ary.p[semid]            // 拿到 kern_ipc_perm*
          ↓ 强转回真正的类型
        sma = (struct sem_array *)perm        // 现在能访问 sems[] 了
          ↓
        执行 P/V，必要时把当前进程挂到 sma->sem_perm 的等待队列
          ↓
        返回用户态
```

`shmat(shmid, ...)` 走的是同一条路，只是最后强转成 `struct shmid_kernel*`，然后去操作 `shm_file`、建立页表映射。

## 这就是 C 语言的"多态"

老师在板书上写的那句：

> **`(msg_queue*)p[0]->other!!  用 C 语言，实现多态！！`**

正是这个意思：

| 面向对象概念 | C 语言实现 |
| :--- | :--- |
| 基类 | `struct kern_ipc_perm`（放在最前面） |
| 子类 | `struct shmid_kernel` / `msg_queue` / `sem_array` |
| 向上转型 | `(struct kern_ipc_perm *)obj` |
| 向下转型 | `(struct shmid_kernel *)perm` |
| 虚函数表 | `struct file_operations *f_op`（函数指针集合） |

## 内核里还有更多同样的套路

这套"公共头放最前 + 强转"的设计，在 Linux 内核里到处都有：

| 结构体 | 公共头 |
| :--- | :--- |
| `struct socket` / `struct file` | `struct file_operations *f_op` |
| `struct ext4_inode_info` | `struct inode vfs_inode`（VFS 层的 inode 放在最后，靠 `container_of` 反推） |
| `struct task_struct` | 通过 `list_head tasks` 挂到全局进程链表 |
| 设备驱动 | `struct cdev` 嵌入到自定义设备结构体里 |

> 补充一个反向技巧：`container_of(ptr, type, member)` 宏。当公共头不在最前面（比如 `inode` 被放在最后），就用已知成员的地址**减去偏移量**反推出整个结构体的地址。这跟"放最前面直接强转"是同一思想的两种实现。

## 为什么要这样设计

1. **代码复用**：权限检查、key 查找、id 分配、`IPC_RMID` 的删除逻辑，只写一遍，三种资源共用。
2. **统一接口**：用户态看到的 `get` / `ctl` 风格一致，学习成本低。
3. **可扩展**：将来加一种新 IPC 类型，只要把 `kern_ipc_perm` 放在第一个成员，就能直接复用整套框架。
4. **数组管理高效**：id 是下标，`O(1)` 定位；柔性数组一次分配，缓存友好。

## 一句话总结

**Linux 用「公共头结构体放在最前面 + C 语言指针强转 + 统一 id 数组」这三招，在纯 C 的环境里实现了面向对象的继承与多态，把共享内存、消息队列、信号量统一管了起来。**
