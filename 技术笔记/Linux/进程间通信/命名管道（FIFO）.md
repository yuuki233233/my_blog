# 命名管道（FIFO）

## 为什么需要命名管道

匿名管道靠 `fork()` 继承文件描述符来共享，所以**只能用于有血缘关系的进程**。
如果两个毫无关系的进程（两个独立的可执行程序）想通信怎么办？

答案：**命名管道（Named Pipe / FIFO）**。
它在**文件系统里有一个真实存在的路径和名字**，任何进程只要知道这个路径，就能 `open` 它，从而指向内核里同一个管道对象。

> 一句话：命名管道 = 用「路径的唯一性」替代「fork 的血缘继承」。

## 创建方式

```bash
# 命令行创建
mkfifo mypipe
ls -l mypipe   # 权限前面会显示 p，表示 pipe
```

```c
#include <sys/types.h>
#include <sys/stat.h>

// 函数创建，mode 受 umask 影响，通常先 umask(0)
int mkfifo(const char *pathname, mode_t mode);
```

## 命名管道与匿名管道的异同

| 对比项 | 匿名管道 `pipe()` | 命名管道 `mkfifo()` |
| :--- | :--- | :--- |
| 是否有名字/路径 | 无 | **有**，在文件系统里可见 |
| 数据存哪 | 内核内存缓冲区 | 内核内存缓冲区（**不落盘**） |
| 能否跨亲缘进程 | 只能靠 fork 继承 fd | **任意进程**知道路径即可 |
| 需要 flush 落盘吗 | 不需要 | **不需要**（管道文件只被打开，不刷新） |
| 读写语义 | 阻塞、原子性、`read` 返回 0 表示写端全关 | **完全一致** |

> 关键认知：虽然 FIFO 在磁盘上有一个"文件"，但它的大小永远是 0，数据一个字节都不会写到磁盘。这个文件只承担"门牌号"的作用。

## 打开规则（阻塞点，最容易踩坑）

- 以 **只读 `O_RDONLY`** 打开 FIFO：**阻塞**，直到有另一个进程以写方式打开它
- 以 **只写 `O_WRONLY`** 打开 FIFO：**阻塞**，直到有另一个进程以读方式打开它
- 加 `O_NONBLOCK`：
  - 只读打开：立刻成功返回
  - 只写打开：立刻失败返回 `ENXIO`（因为没读端，写了也没意义）

## 实战：用命名管道实现文件拷贝

读取文件，写入命名管道
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <fcntl.h>

#define ERR_EXIT(m) \
do \
{ \
    perror(m); \
    exit(EXIT_FAILURE); \
} while(0)

int main(int argc, char *argv[])
{
    // 创建名为 tp 的命名管道
    if (mkfifo("tp", 0644) == -1) ERR_EXIT("mkfifo");

    // 以读的方式打开 abc 文件
    int infd;
    infd = open("abc", O_RDONLY);
    if (infd == -1) ERR_EXIT("open");
    
    // 以写的方式打开管道 tp
    int outfd;
    outfd = open("tp", O_WRONLY)
    if(outfd == -1) ERR_EXIT("open");

    char buf[1024];
    int n;
    // 数据流动 (文件->缓冲区->管道)
    while((n = read(infd, buf, 1024)) > 0)
    {
        write(outfd, buf, n);
    }
    close(infd);
    close(outfd);
  
    return 0;
}
```

读取管道，写入目标文件
```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <fcntl.h>

#define ERR_EXIT(m) \
do \
{ \
    perror(m); \
    exit(EXIT_FAILURE); \
} while(0)

int main(int argc, char *argv[])
{
    int outfd;
    outfd = open("abc.bak", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (outfd == -1) ERR_EXIT("open");

    int infd;
    infd = open("tp", O_RDONLY);
    if (infd == -1) ERR_EXIT("open");

    char buf[1024];
    int n;
    // 数据流动 (管道->缓冲区->备份文件)
    while((n = read(infd, buf, 1024)) > 0)
    {
        write(outfd, buf, n);
    }
    close(infd);
    close(outfd);
    // 删除管道文件
    unlink("tp");

    return 0;
}
```

## 实战：Server / Client 通信

**server.cc** —— 创建 FIFO，阻塞等客户端，循环读

```cpp
#include <iostream>
#include <cstring>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FIFO_FILE "./myfifo"

int main() {
    umask(0);
    // 已存在会失败，忽略该错误即可
    if (mkfifo(FIFO_FILE, 0666) == -1 && errno != EEXIST) {
        perror("mkfifo");
        return 1;
    }

    // 1. 只读打开，阻塞等待写端出现
    int fd = open(FIFO_FILE, O_RDONLY);
    if (fd < 0) { perror("open"); return 2; }

    char buffer[1024];
    while (true) {
        // 2. read 返回 0 表示所有写端都已关闭
        ssize_t n = read(fd, buffer, sizeof(buffer) - 1);
        if (n > 0) {
            buffer[n] = 0;   // 手动补字符串结束符，否则 cout 会打印越界垃圾
            std::cout << "client say# " << buffer << std::endl;
        } else if (n == 0) {
            std::cout << "client quit, server exit" << std::endl;
            break;
        } else {
            perror("read");
            break;
        }
    }

    close(fd);
    unlink(FIFO_FILE);   // 删除管道文件
    return 0;
}
```

**client.cc** —— 打开同一个路径写入

```cpp
#include <iostream>
#include <string>
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("FIFO_FILE", O_WRONLY);
    if (fd < 0) { perror("open"); return 1; }

    std::string line;
    while (true) {
        std::cout << "Please Enter# ";
        std::getline(std::cin, line);
        if (line == "quit") break;
        write(fd, line.c_str(), line.size());
    }

    close(fd);
    return 0;
}
```

**几个必踩的坑**

1. `read` 读到数据后**不会自动加 `\0`**，必须写 `buffer[n] = 0;`，否则 `cout << buffer` 会越界打印乱码。
2. `read` 的第三个参数写成 `sizeof(buffer) - 1`，专门给 `\0` 留一个字节，保证 `buffer[n] = 0` 永不越界。
3. 服务端退出前要 `unlink`，否则下次 `mkfifo` 会因为文件已存在而失败（所以上面用 `errno != EEXIST` 放行）。
4. 客户端 `Ctrl+C` 退出后，`read` 返回 0，服务端要能识别并正常退出，否则会陷入忙等待。

## 进阶：用两根管道实现链式命令

`cat file | grep xxx | wc -l` 这种三级流水线，本质是：

- 管道 A：连接 `cat` 的写端 → `grep` 的读端
- 管道 B：连接 `grep` 的写端 → `wc` 的读端

三个进程、两根命名管道，配合 `dup2` 把管道的 fd 重定向到各自的标准输入/输出，就能把任意命令串成流水线。这也是 shell 实现 `|` 的原理。
