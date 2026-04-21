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

mode:利用到umaks计算权限

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

## 读文件

## open函数