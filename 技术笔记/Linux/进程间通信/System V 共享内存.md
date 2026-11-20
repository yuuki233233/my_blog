# System V 共享内存

## 为什么共享内存是最快的 IPC

管道的通信路径：

```
进程A用户态 → write() → 内核缓冲区 → read() → 进程B用户态
```

数据**拷贝 2 次**，还要**陷入内核 2 次**（用户态 <-> 内核态切换开销大）。

共享内存的路径：

```
进程A虚拟地址 ──┐
               ├── 页表 ──→ 同一块物理内存
进程B虚拟地址 ──┘
```

- 映射建立之后，读写就是**普通的内存访问**，指令级别直接完成
- **零次系统调用、零次数据拷贝** -> 所以最快

> 代价：内核完全不管你们读写的节奏，**没有任何同步与互斥保护**。

## key 与 shmid 的区别（最容易混淆）

| 概念        | 类型      | 面向谁   | 作用                                | 比喻          |
| :-------- | :------ | :---- | :-------------------------------- | :---------- |
| **key**   | `key_t` | 用户/内核 | 进程间**约定**的标识，用于 `shmget` 查找或创建    | 约定的**暗号**   |
| **shmid** | `int`   | 用户/内核 | 内核返回的**句柄**，用于后续 `shmat`/`shmctl` | 前台给的**房间号** |
| **路径**    | `char*` | 用户    | `ftok` 的参数，用来**算出** key 的种子       | 约定暗号时参考的那本书 |

两个进程怎么"找到同一个 key"？

```c
// 进程A 和 进程B 都用同一个路径 + 同一个 proj_id
key_t key = ftok("/tmp/myfile", 0x66);
```

因为输入相同，`ftok` 的算法输出必然相同 → 两边算出**同一个 key**。
注意：`ftok` 用的那个路径文件**只是为了算哈希**，和共享内存本身没有任何物理关系，共享内存里写的数据也绝不会进这个文件。

## 核心 API 五件套

```c
#include <sys/ipc.h>
#include <sys/shm.h>

key_t ftok(const char *pathname, int proj_id);              // 1. 生成 key

int   shmget(key_t key, size_t size, int shmflg);           // 2. 创建/获取
void *shmat(int shmid, const void *shmaddr, int shmflg);    // 3. 挂载（映射）
int   shmdt(const void *shmaddr);                           // 4. 卸载（解除映射）
int   shmctl(int shmid, int cmd, struct shmid_ds *buf);     // 5. 控制/删除
```

### shmget 创建/获取

```bash
# 功能：用来创建共享内存
int shmget(key_t key, size_t size, int shmflg)
```

| 命令       | 说明                             |
| -------- | ------------------------------ |
| `key`    | 共享内存段名字                        |
| `size`   | 共享内存大小                         |
| `shmflg` | 标志位                            |
| `返回值`    | 成功返回一个非负整数，即该共享内存段的标识码；失败返回 -1 |

**shmflg（标志位）**：
- `IPC_CREAT`：不存在就创建
- `IPC_EXCL`：配合 `IPC_CREAT`，已存在就报错（保证拿到的是新建的）
- `0666`：权限位
- 客户端只需 `shmget(key, size, IPC_CREAT)`（不加 `IPC_EXCL`，拿到已有的）

### shmat 挂载（映射）

```bash
# 功能：将共享内存段连接到进程地址空间
void *shmat(int shmid, const void *shmaddr, int shmflg)
```

| 命令          | 说明                          |
| ----------- | --------------------------- |
| ``shmid``   | 共享内存标识                      |
| ``shmaddr`` | 指定连接的地址                     |
| `shmflg`    | 标志位                         |
| `返回值`       | 成功返回一个指向共享内存第一个节的指针；失败返回 -1 |

**shmaddr**：
- NULL：核心自动选择一个地址
- 不为 NULL
	- `shmflg` 无 `SHM_RND` 标记，则以 `shmaddr` 为连接地址
	- `shmflg` 设置了 `SHM_RND` 标记，则连接的地址会自动向下调整为 `SHMLBA` 的整数倍。公式：`shmaddr - (shmaddr % SHMLBA)`
- `shmflg = SHM_RDONLY` 表示连接操作用来只读共享内存
**shmflg（标志位）**：`SHM_RND` 和 `SHM_RDONLY`

### shmdt 卸载（解除映射）

```bash
# 功能：将共享内存段与当前进程脱离
int shmdt(const void *shmaddr);
```
`shmaddr`：由 `shmat` 所返回的指针
返回值：成功返回 0；失败返回 -1
注意：将共享内存段与当前进程脱离不等于删除共享内存

### shmctl 控制/删除

 ```bash
 # 功能：⽤于控制共享内存
 int shmctl(int shmid, int cmd, struct shmid_ds *buf);
 ``` 

| 命令      | 说明                         |
| ------- | -------------------------- |
| `shmid` | `shmget` 返回的共享内存标识         |
| `cmd`   | 标志位                        |
| `buf`   | 指向一个保存着共享内存的模式状态和访问权限的数据结构 |
| 返回值     | 成功返回 0；失败返回 -1             |

**cmd（标志位）**：
 - `IPC_STAT`：把 shmid_ds 结构中的数据设置为共享内存的当前关联值
 - `IPC_SET`：在进程由足够权限的前提下，把共享内存的当前关联值设置为 shmid_ds 数据结构中给出的值
 - `IPC_RMID`：删除共享内存段

## 完整封装（comm.hpp）

```cpp
#pragma once

#include <iostream>
#include <cstring>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>

const char *PATH_NAME = "/tmp";
const int   PROJ_ID   = 0x66;
const int   SHM_SIZE  = 4096;

key_t GetKey() {
    key_t key = ftok(PATH_NAME, PROJ_ID);
    if (key == -1) { std::cerr << "ftok error" << std::endl; exit(1); }
    return key;
}

// Server：创建（IPC_EXCL 保证是新建的）
int CreateShm() {
    int shmid = shmget(GetKey(), SHM_SIZE, IPC_CREAT | IPC_EXCL | 0666);
    if (shmid == -1) { std::cerr << "shmget create error" << std::endl; exit(2); }
    return shmid;
}

// Client：获取（不带 IPC_EXCL）
int GetShm() {
    int shmid = shmget(GetKey(), SHM_SIZE, IPC_CREAT);
    if (shmid == -1) { std::cerr << "shmget get error" << std::endl; exit(3); }
    return shmid;
}

void *AttachShm(int shmid) {
    void *addr = shmat(shmid, nullptr, 0);
    if (addr == (void *)-1) { std::cerr << "shmat error" << std::endl; exit(4); }
    return addr;
}

void DetachShm(void *addr) {
    if (shmdt(addr) == -1) { std::cerr << "shmdt error" << std::endl; exit(5); }
}

void DeleteShm(int shmid) {
    if (shmctl(shmid, IPC_RMID, nullptr) == -1) {
        std::cerr << "shmctl delete error" << std::endl; exit(6);
    }
}
```

**server.cc**

```cpp
int main() {
    int shmid = CreateShm();
    char *addr = (char *)AttachShm(shmid);

    strcpy(addr, "Hello, I am Server! This is shared memory.");

    sleep(5);            // 简易同步：等服务端读完（真实项目要用信号量）
    DetachShm(addr);
    DeleteShm(shmid);    // 谁创建谁删除
    return 0;
}
```

**client.cc**

```cpp
int main() {
    int shmid = GetShm();
    char *addr = (char *)AttachShm(shmid);

    std::cout << "read: " << addr << std::endl;   // 像读普通指针一样

    DetachShm(addr);
    return 0;
}
```

## 致命缺陷：没有同步机制

共享内存是一块**裸内存**，内核不参与你的读写过程。所以会出现：

| 灾难 | 现象 |
| :--- | :--- |
| 写一半被读 | Server 写到 `"AA BB"` 就被切走，Client 读到 `"AA B"`（残缺） |
| 覆盖重写 | Client 还没读，Server 又写了一遍，旧数据丢失 |
| 读到脏数据 | Client 读到了上一轮的残留内容 |

**这就是为什么共享内存必须搭配「信号量」使用。** 共享内存负责"快"，信号量负责"安全"。

## 生命周期：随内核

这点和其他 IPC 都不同，必须记住：

- **共享内存不随进程结束而释放**
- 进程全部退出后，`shm_nattch`（挂载计数）变为 0，但物理内存仍在
- 必须显式 `shmctl(shmid, IPC_RMID, NULL)`，或重启系统才会消失
- 所以大量使用 System V 共享内存的老系统，常见"内存泄漏"

**运维命令**

```bash
ipcs -m              # 查看所有共享内存段
ipcrm -m <shmid>     # 手动删除指定段
ipcs -a              # 查看全部 IPC 资源（shm + msg + sem）
```

## 与动态库加载的类比

两者在底层是同一套机制：**让不同进程的页表指向同一块物理内存**。

| 对比项 | System V 共享内存 | 动态库 `.so` 映射 |
| :--- | :--- | :--- |
| 是否有磁盘文件 | **无**（tmpfs 上的匿名文件） | **有**（磁盘上的 .so） |
| 是否可写 | 可读写 | 代码段只读 |
| 谁来映射 | `shmat` | 动态链接器 `mmap` |
| 目的 | 传数据 | 省内存、共享代码 |
