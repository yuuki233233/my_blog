# System V 信号量

## 并发编程的四个基石概念

在写任何"多执行流访问共享数据"的代码之前，必须先把这四个词分清楚。

| 概念 | 定义 | 例子 |
| :--- | :--- | :--- |
| **共享资源** | 多个执行流（进程/线程）都能看到的同一份资源 | 共享内存、全局变量、文件 |
| **临界资源** | **被保护起来的**共享资源 | 加了锁的共享内存 |
| **临界区** | 代码中**访问临界资源的那段程序** | `strcpy(shmaddr, msg)` 这一行 |
| **原子性** | 操作要么全部完成，要么完全不执行，中间不被打断 | 加锁这个动作本身 |

> **老师反复强调的一句话**：
> 所谓的"保护共享资源"，**本质是保护访问共享资源的那段代码**。
> 内核没法给一块内存挂一把物理锁，它只能控制"能不能进入那段代码"。

## 灾难是怎么发生的

```
Server: 想写 "AA BB CC"
        写到 "AA BB" → 时间片耗尽，被切走
Client: 此时读到 "AA B"        ← 数据残缺
```

只要有：**① 共享资源 + ② 多个执行流 + ③ 同时访问**，就必然出现**数据不一致**。

## 信号量是什么

**信号量（Semaphore）本质上是一个计数器**，由内核维护，用来表示**当前还有多少个资源可用**。

### 电影院模型

- 电影院（共享资源）有 16 个座位
- 信号量初始值 = 16（剩余座位数）
- 来一个人 → 计数器 `-1`（**买票 = 预订资源**）
- 走一个人 → 计数器 `+1`（**退票 = 归还资源**）
- 计数器为 0 时，再来的人只能**排队等**（阻塞）

要防的两件事：
1. **超卖**（票卖多了）→ 计数器必须准确，操作必须原子
2. **一票多人**（两个进程同时以为自己拿到了资源）→ 必须互斥

### P 操作与 V 操作

| 操作 | 别名 | 动作 | 效果 |
| :--- | :--- | :--- | :--- |
| **P** | 申请资源 | `sem--` | 若 `sem < 0`，进程**阻塞挂起** |
| **V** | 释放资源 | `sem++` | 若有等待者，**唤醒**它 |

> P 来自荷兰语 Proberen（尝试），V 来自 Verhogen（增加）。这两个操作**必须是原子的**，靠内核的原子指令 + 自旋锁保证。

### 二元信号量 = 互斥锁

| 类型 | 初始值 | 用途 |
| :--- | :--- | :--- |
| **多元信号量** | `sem > 1` | 管理多个相同资源（如 16 个座位） |
| **二元信号量** | `sem = 1` | **互斥锁（Mutex）**，只允许一个执行流进入 |

二元信号量的状态机：

```
sem = 1  →  P操作  →  sem = 0  （有人占用了）
sem = 0  →  再来P  →  sem = -1 （阻塞等待，-1 表示有 1 个在等）
sem = -1 →  V操作  →  sem = 0  （唤醒一个等待者，但它还没真正进入）
```

> 注意：`sem` 为负数时，其绝对值 = 正在等待的进程数。这是判断死锁/饥饿的重要线索。

## 关键认知：信号量自己也是共享资源

这是最容易被忽略的一点，也是老师板书上的"灵魂拷问"：

1. **信号量自己也是共享资源** → 所以它必须由内核维护（`semget` 创建），进程 A 和 B 通过同一个 `key` 找到**同一个信号量**。
2. **对信号量的 P/V 操作也必须原子** → 如果 A 在执行 `sem--` 的过程中被切走，B 也执行 `sem--`，两人会同时认为自己拿到了锁，锁直接失效。所以内核必须用原子指令或自旋锁保护信号量本身。

## 核心 API

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);                        // 创建/获取
int semctl(int semid, int semnum, int cmd, ...);                     // 控制
int semop(int semid, struct sembuf *sops, size_t nsops);             // P/V 操作
```

### semget：一次可以创建一组

```c
// nsems = 1 表示创建 1 个信号量；也可以一次创建 N 个（信号量集）
int semid = semget(GetKey(), 1, IPC_CREAT | IPC_EXCL | 0666);
```

### semctl：命令多，只需记 2 个常用

`semctl` 的第 4 个参数是个 `union semun`，因为不同命令需要的参数类型不同（`int` / 结构体 / 数组），C 语言没有重载，只能用联合体：

```c
union semun {
    int              val;    // SETVAL 用
    struct semid_ds *buf;    // IPC_STAT / IPC_SET 用
    unsigned short  *array;  // GETALL / SETALL 用
};
```

| 命令 | 作用 | 典型用法 |
| :--- | :--- | :--- |
| **`SETVAL`** | 设置某信号量的初值 | `semctl(semid, 0, SETVAL, 1)` 初始化成互斥锁 |
| **`IPC_RMID`** | 删除整组信号量 | `semctl(semid, 0, IPC_RMID)` 防泄漏 |
| `IPC_STAT` | 获取状态到 `semid_ds` | 查看 `sem_nsems`、`sem_otime` |
| `IPC_SET` | 修改权限 | 改 `sem_perm.mode` |

> 只记前两个就够用 99% 的场景。

### semop：P/V 的真正执行者

```c
struct sembuf {
    unsigned short sem_num;  // 第几个信号量（从 0 开始）
    short          sem_op;   // P 填 -1，V 填 +1
    short          sem_flg;  // 0 表示阻塞；IPC_NOWAIT 表示非阻塞
};
```

一次 `semop` 可以传一个 `sembuf` 数组，内核会**原子地把整个数组的操作一起完成**——这能做"同时申请多个资源"，避免死锁。

## 封装成 RAII 风格的锁

```cpp
class Sem {
public:
    Sem() : _semid(Create()) { Init(1); }

    void P() { Op(-1); }        // 加锁
    void V() { Op(1);  }        // 解锁
    ~Sem() { semctl(_semid, 0, IPC_RMID); }   // 自动删除，防泄漏

private:
    int Create() {
        int semid = semget(GetKey(), 1, IPC_CREAT | IPC_EXCL | 0666);
        if (semid == -1) { std::cerr << "semget error" << std::endl; exit(1); }
        return semid;
    }

    void Init(int val) {
        union semun arg;
        arg.val = val;
        semctl(_semid, 0, SETVAL, arg);
    }

    void Op(short op) {
        struct sembuf buf;
        buf.sem_num = 0;
        buf.sem_op  = op;
        buf.sem_flg = 0;        // 阻塞方式
        if (semop(_semid, &buf, 1) == -1) {
            std::cerr << "semop error" << std::endl; exit(2);
        }
    }

    int _semid;
};
```

### 配合共享内存使用

```cpp
// server
Sem sem;                       // 信号量初值 1
char *addr = (char *)AttachShm(shmid);

sem.P();                       // 加锁，进入临界区
strcpy(addr, "AA BB CC");
sem.V();                       // 解锁，离开临界区

// client
sem.P();                       // 如果 server 还在写，这里就阻塞住
std::cout << addr << std::endl;
sem.V();
```

这样就实现了"**写的时候别人不能读，读的时候别人不能写**"，彻底消灭 `AA B` 这种残缺数据。

## 生命周期与清理

和共享内存一样，**信号量随内核存在**，进程退出不释放：

```bash
ipcs -s                # 查看所有信号量集
ipcrm -s <semid>       # 手动删除
```

> 上面 `Sem` 的析构函数里调 `IPC_RMID` 就是为了避免这一点。生产环境如果进程被 `kill -9`，析构不会执行 → 依然会残留，需要运维脚本定期清理。

## 一句话总结

**共享内存负责"快"（零拷贝），信号量负责"安全"（P/V 互斥）。信号量本身也是一个由内核维护、必须原子操作的共享计数器，二元信号量就是互斥锁。**
