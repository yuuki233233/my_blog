## 系统文件接口
手册：`man 2 open`

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

pathname: 要打开或创建的目标文件

flags: 多参数选项，想做什么就传什么参数
参数: 
1. O_RDONLY: 只读打开
2. O_WRONLY: 只写打开
3. O_RDWR: 读，写打开 (1.2.3中只能选一个)
4. O_CREAT: 创建新文件，需要用到mode选项，来指定文件的权限
5. O_APPEND: 追加
6. O_TRUNC: 清空文件数据

mode:权限位，新文件必须写

返回值:
成功: 新打开的文件描述符
失败: -1
```
`open`：
1. 文件存在，用第一个
2. 文件不存在，用第二个，且必须传 mode 值

`write` `read` `close` `lseek`，与C文件接口一致

## 标志位（位图）

```c

```

## 写文件

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```
将 `const void *buf` 写入到 `int fd`

系统层面：`const void *buf`
- 字符类型以字符写入
- 整数类型以二进制写入
- 想以整数传入，就得格式化为字符类型

语言层面：
- 语言的 fwrite() 底层调用的是系统的 write()
- fwrite() 做了些改变，字符/整形都以字符串写入

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
    umask(0);

    // 追加
    int fd = open("log.txt", O_CREAT | O_WRONLY | O_APPEND, 0666);
    
    // 清空
    // int fd = open("log.txt", O_CREAT | O_WRONLY | O_TRUNC, 0666);

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
        write(fd, msg, strlen(msg)); // 写入
    }

    close(fd);

    return 0;
}
```

- 整数形式改为字符串输入（直接输入整数，读取为乱码）
```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
    umask(0);

    // 追加
    //int fd = open("log.txt", O_CREAT | O_WRONLY | O_APPEND, 0666);
    
    // 插入 
    int fd = open("log.txt", O_CREAT | O_WRONLY | O_TRUNC, 0666);

    if(fd < 0)
    {
        perror("open");
        return 1;
    }
    printf("fd: %d\n", fd);

    //const char *msg = "abcd";
    int cnt = 1;
    int a = 1234567;
    while(cnt--)
    {
        // 整数->字符
        char buffer[16];
        snprintf(buffer, sizeof(buffer), "%d", a); // 格式化为字符
        write(fd, buffer, strlen(buffer));
    }

    close(fd);

    return 0;
}
```

## 读文件

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
```
- 将读取到得文件写入 `void *buf`
- 失败：返回值<0
- 读文件不需要 `O_CREAT 创建文件` 和 `O_TRUNC 清空`，只需 `O_RDONLY 

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
    umask(0);

    //int fd = open("log.txt", O_CREAT | O_WRONLY | O_APPEND, 0666); 
    //int fd = open("log.txt", O_CREAT | O_WRONLY | O_TRUNC, 0666);
    int fd = open("log.txt", O_RDONLY);

    if(fd < 0)
    {
        perror("open");
        return 1;
    }
    printf("fd: %d\n", fd);

    while(1)
    {
        // 写入到buffer并打印buffer
        char buffer[64];
        int n = read(fd, buffer, sizeof(buffer)-1);
        if(n > 0) // 读取成功
        {
            buffer[n] = 0;
            printf("%s", buffer);
        }
        else if(n == 0) // 结束
        {
            break;
        }
    }

    close(fd);

    return 0;
}
```

## open函数返回值

>系统调用
>`open` `close` `read` `write` `lseek` 都属于系统接口

>库函数
>`fopen` `fclose` `fread` `fwrite` C标准库函数接口

上面库函数系列接口，都是对系统调用的二次封装