# 系统文件 IO：从底层出发

虽然 C 库函数很好用，但在底层，真正和操作系统打交道的是系统调用。`man 2 open` 可以查看底层接口

## `open` 函数解析

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

// 文件存在用一接口，否则二接口（必须传 mode 值）
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

pathname: 打开或创建的目标文件路径

flags: 标志位
参数: 
1. O_RDONLY: 只读
2. O_WRONLY: 只写
3. O_RDWR: 读，写 (1.2.3中只能选一个)
4. O_CREAT: 不存在则创建，需要用到 mode 选项，来指定文件的权限
5. O_APPEND: 追加
6. O_TRUNC: 清空
多个标志位通过 按位或 | 组合

mode:权限位。如果使用了 O_CREAT 创建新文件，就必须传入这个参数（如 0666）
通常配合 umask(0) 使用，避免权限被系统默认屏蔽

返回值:
成功: 新打开的文件描述符
失败: -1
```

- **pathname**：目标文件或创建的目标文件路径
- **flags**：标志位。常用的有 `O_RDONLY`（只读）、`O_WRONLY`（只写）、`O_RDWR`（读写）、`O_CREAT`（不存在则创建）、`O_TRUNC`（清空）、`O_APPEND`（追加）。多个标志位通过**按位或 `|`** 组合
- **mode**：权限位。如果使用了 `O_CREAT` 创建新文件，就必须传入这个参数（如 `0666`）。通常配合 `umask(0)` 使用，避免权限被系统默认屏蔽
- **返回值**：成功返回文件描述符（一个整数），失败返回 `-1`

>[!note] **为什么用 `|` 传参？**  
因为标志位本质是位图（宏定义如 `#define ONE 0001`, `#define TWO 0002`）。`flags & ONE` 就能判断是否包含某个标志，这极大节省了参数传递的空间

## `write` 与 `read`

```c
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t count);
ssize_t read(int fd, void *buf, size_t count);
```

### 实战：系统调用版写文件

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```
将缓冲区（`const void *buf`） 写到文件（ `int fd`）

- 以写形式打开文件，清空输入 `int fd = open("log.txt", O_CREAT | O_WRONLY | O_TRUNC, 0666);`
- 以写形式打开文件，追加输入 `int fd = open("log.txt", O_CREAT | O_WRONLY | O_APPEND, 0666);`
```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    umask(0); // 清除umask，保证权限0666生效
 
    // 打开文件：创建 + 只写 + 清空，权限 0666
    int fd = open("log.txt", O_CREAT | O_WRONLY | O_TRUNC, 0666);
    if(fd < 0)
    {
        perror("open");
        return 1;
    }
    printf("fd: %d\n", fd);

    const char *msg = "abcd";
    int cnt = 1;
    while(cnt--)
    {
        write(fd, msg, strlen(msg)); // 写入数据
    }

    close(fd); // 关闭文件

    return 0;
}
```

- 整数形式改为字符串输入（直接输入整数，读取为乱码）
```c
 int cnt = 1;
    int a = 1234567;
    while(cnt--)
    {
        // 整数->字符
        char buffer[16];
        snprintf(buffer, sizeof(buffer), "%d", a); // 格式化为字符
        write(fd, buffer, strlen(buffer));
    }
```

>[!warning]
>**注意坑点**：系统调用 `write` 写入的是**原始字节**。如果你强行传一个 `int` 变量的地址进去，它会把内存的二进制字节写进文件，导致你 `cat` 的时候看到乱码。想写入数字，必须用 `snprintf` 格式化成字符串

### 实战：系统调用版读文件

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
```
将读取到的文件 `int fd` 得文件写入缓冲区 `void *buf`

- 以读形式打开文件： `int fd = open("log.txt", O_RDONLY);`
```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    int fd = open("log.txt", O_RDONLY); // 读文件只需 O_RDONLY

    if(fd < 0)
    {
        perror("open");
        return 1;
    }
    printf("fd: %d\n", fd);

    // 写入到缓冲区 buffer 并打印 buffer
	char buffer[64];
    while(1)
    {
        int n = read(fd, buffer, sizeof(buffer)-1);
        if(n > 0) // 读取成功
        {
            buffer[n] = 0; // 添加字符串结束符
            printf("%s", buffer);
        }
        else if(n == 0)
        {
            break;// 读到文件末尾
        }
    }

    close(fd);

    return 0;
}
```