# System V 消息队列

## 是什么

**消息队列（Message Queue）** 是内核维护的一个**链表**，链表的每个节点是一条"消息"，消息带一个**类型标签（`long mtype`）**。

- 发送方：把消息挂到链表尾部
- 接收方：可以**按类型挑选**要收哪一条

> 一句话：**带类型标签、由内核帮忙排队、自带同步的共享内存**。

## 核心特色：类型（mtype）

这是它区别于管道最大的地方。

管道是**无格式字节流**：写 `AA BB CC`，读出来就是 `AA BB CC` 顺序。
消息队列允许打标签：

```
进程A → [type=1] 订单消息
进程B → [type=2] 日志消息
        ↓
   ┌─────────┬─────────┬─────────┐
   │ type=1  │ type=2  │ type=1  │    内核维护的链表
   └─────────┴─────────┴─────────┘
        ↓
进程C 调用 msgrcv(..., type=1, ...) → 只拿到 type=1 的两条
```

**用途**：一个服务端（进程C）用同一个队列，接收多个客户端的不同类型请求，互不串味。这就是"多路复用"。

## 内核如何组织

内核中对应结构体 `struct msg_queue`，它包含：

- `kern_ipc_perm q_perm`：公共头部（权限、key，见 [[内核如何统一管理 IPC 资源]]）
- `struct list_head q_messages`：消息链表的头
- `struct list_head q_receivers`：正在等待接收的进程队列（睡眠队列）
- `struct list_head q_senders`：正在等待发送的进程队列（队列满时）
- `long q_cbytes`：当前队列中总字节数
- `long q_qnum`：当前消息条数

> 这就是操作系统"**先描述，再组织**"的又一次体现。

## 核心 API

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgget(key_t key, int msgflg);                                   // 创建/获取
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);    // 发送
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg); // 接收
int msgctl(int msqid, int cmd, struct msqid_ds *buf);                 // 控制/删除
```

### 消息结构体：必须是 long 开头

```c
struct msgbuf {
    long mtype;      // 必须 > 0，且必须是第一个成员！
    char mtext[100]; // 真正的数据
};
```

**为什么强制 `long` 打头？**
内核内部用 `container_of` 之类的技巧，通过消息节点的地址反推回去定位 `mtype`。所以这个字段的位置和类型是硬性约定，写成 `int` 会直接跑飞。

## msgrcv 的 msgtyp 参数（重点）

| `msgtyp` 取值 | 含义                                 |
| :---------- | :--------------------------------- |
| `== 0`      | 取队列中**第一条**消息（不管类型），最常用            |
| `> 0`       | 取**第一条类型等于 msgtyp** 的消息            |
| `< 0`       | 取类型**小于等于 `msgtyp` 中最小**的那条（优先级语义） |

第三种用法可以做出"优先级队列"效果。

## 完整实战

**comm.hpp**

```cpp
#pragma once
#include <iostream>
#include <cstring>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <unistd.h>

const char *PATH_NAME = "/tmp";
const int   PROJ_ID   = 0x77;

struct MsgBuf {
    long mtype;
    char mtext[128];
};

key_t GetKey() {
    key_t key = ftok(PATH_NAME, PROJ_ID);
    if (key == -1) { std::cerr << "ftok error" << std::endl; exit(1); }
    return key;
}

int CreateMsgQueue() {
    int msqid = msgget(GetKey(), IPC_CREAT | IPC_EXCL | 0666);
    if (msqid == -1) { std::cerr << "msgget create error" << std::endl; exit(2); }
    return msqid;
}

int GetMsgQueue() {
    int msqid = msgget(GetKey(), IPC_CREAT);
    if (msqid == -1) { std::cerr << "msgget get error" << std::endl; exit(3); }
    return msqid;
}

void DeleteMsgQueue(int msqid) {
    if (msgctl(msqid, IPC_RMID, nullptr) == -1) {
        std::cerr << "msgctl error" << std::endl; exit(4);
    }
}
```

**server.cc** —— 创建队列，按类型读

```cpp
int main() {
    int msqid = CreateMsgQueue();
    std::cout << "server msqid = " << msqid << std::endl;

    MsgBuf msg;
    while (true) {
        // 只收 type == 1 的消息
        ssize_t n = msgrcv(msqid, &msg, sizeof(msg.mtext), 1, 0);
        if (n > 0) {
            std::cout << "recv type=" << msg.mtype
                      << " data=" << msg.mtext << std::endl;
            if (strcmp(msg.mtext, "quit") == 0) break;
        } else {
            perror("msgrcv");
            break;
        }
    }

    DeleteMsgQueue(msqid);
    return 0;
}
```

**client.cc** —— 发送带类型的消息

```cpp
int main() {
    int msqid = GetMsgQueue();

    MsgBuf msg;
    msg.mtype = 1;                       // 打标签
    strcpy(msg.mtext, "hello from client");
    msgsnd(msqid, &msg, sizeof(msg.mtext), 0);

    msg.mtype = 1;
    strcpy(msg.mtext, "quit");
    msgsnd(msqid, &msg, sizeof(msg.mtext), 0);

    return 0;
}
```

## 自带同步（不用额外加锁）

因为数据是"内核帮忙排队"的，所以同步天然成立：

- 队列空 → `msgrcv` **阻塞**，进程睡眠，不占 CPU
- 队列满（受 `msgmnb` 限制）→ `msgsnd` **阻塞**
- 多个发送者并发发送 → 每条消息**整体入队**，不会像共享内存那样"写一半被读"

> 这也是它比共享内存"安全但慢"的原因：安全来自内核中介，慢也来自内核中介（两次拷贝 + 系统调用）。

## 为什么现在不常用了

| 问题 | 说明 |
| :--- | :--- |
| 限制多 | 单条 `msgmax`、队列总容量 `msgmnb`、系统总队列数都有上限，超了直接失败 |
| 性能平庸 | 慢于共享内存，复杂于管道 |
| 生命周期难管 | 随内核存在，忘了删就泄漏 |
| 有更好的替代 | POSIX 消息队列（`mq_open`，像文件一样操作）、Socket、ZeroMQ / RabbitMQ |

## 对比总结

| 特性 | 匿名管道 | 命名管道 | 共享内存 | 消息队列 |
| :--- | :--- | :--- | :--- | :--- |
| 数据拷贝次数 | 2 | 2 | **0** | 2 |
| 自带同步 | 有 | 有 | **无** | 有 |
| 支持类型 | 无 | 无 | 自行定义 | **有 `mtype`** |
| 生命周期 | 随进程 | 随进程 | 随内核 | 随内核 |
| 速度 | 中 | 中 | **最快** | 慢 |

> 记住一句话就够应付绝大多数场景：**消息队列 = 带类型的、内核帮忙排队的、不用自己加锁的共享内存。**
