>**作者**：yuuki233233
>**目标**：德国 CS 本科 + 特斯拉软件工程师
>**适用人群**：大一/自学者，想快速上手 Linux 命令行 + 搞懂操作系统底层逻辑

# 一、 重新认识“文件”

### 1.1 狭义与广义的理解

*   **狭义理解**：文件就存在磁盘里。磁盘是外设（既是输入也是输出设备），所以对文件的所有操作，本质上都是对外设的**输入/输出（I/O）**
*   **广义理解**：**Linux 下一切皆文件**。键盘、显示器、网卡甚至进程，在 Linux 看来都可以被抽象成文件来操作

### 1.2 文件到底是什么？

*   **文件 = 属性（元数据） + 内容**。哪怕是一个 0KB 的空文件，它也会占用磁盘空间，因为它的“属性”（如创建时间、权限、inode）需要被存储
*   **系统角度**：对文件的操作，本质是**进程**对文件的操作；而磁盘的管理者是**操作系统**。文件的读写不能直接越过 OS，必须通过文件相关的**系统调用接口**来实现

> [!note] **C语言/C++的库函数（如 `fopen`, `fread`）只是对系统调用接口的二次封装**，为了方便程序员使用。

---


# 二、 C文件接口回顾

C 语言默认会打开三个输入输出流（类型均为 `FILE*`）：
*   `stdin`：标准输入（键盘，对应文件描述符 0）
*   `stdout`：标准输出（显示器，对应文件描述符 1）
*   `stderr`：标准错误（显示器，对应文件描述符 2）

### 2.1 文件的打开方式（mode）

*   `r` (read)：只读打开。文件必须存在
*   `w` (write)：只写打开。文件存在则清空（截断为 0 字节），不存在则创建
*   `a` (append)：追加写。文件存在则在末尾追加，不存在则创建
*   加上 `+` 号（如 `r+`, `w+`, `a+`）表示以读写方式打开

### 2.2 基础实战

#### 1. fwrite 接口

```c
#include <stdio.h>
#include <string.h>

int main() {
    // 以写方式打开文件
    FILE *fp = fopen("myfile", "w");
    if (!fp) {
        perror("fopen");
        return 1;
    }
    
    const char *msg = "hello world\n";
    int count = 5;
    // 写入5次
    while (count--) {
        fwrite(msg, strlen(msg), 1, fp);
    }
    
    fclose(fp);
    return 0;
}
```

如果想把信息输入到显示器，有三种常见方法：
```c
// const char *msg = "hello world!\n";
fwrite(msg, strlen(msg), 1, stdout);
printf("hello printf\n");
fprintf(stdout, "hello fprintf\n");
```

>[!note] **系统怎么知道你的程序在哪个路径下创建了文件？**  
可以通过 `ls /proc/[进程id] -l` 查看，其中 `cwd` 指向当前进程的运行目录（Current Working Directory）。`fopen("myfile", "w")` 默认在当前 `cwd` 下创建

#### 2. fread 接口

```c
#include <stdio.h>
#include <string.h>

int main()
{
	FILE *fp = fopen("myfile", "w");
	if(!fp)
	{
		printf("fopen error!\n");
		return 1;
	}
	
	char buf[1024];
	const char *msg = "hello world\n";
	
	while(1)
	{
		size_t s = fread(buf, 1, strlen(msg), fp);
		if(s > 0)
		{
			buf[s] = 0;
			printf("%s", buf);
		}
		if(feof(fp))
		{
			break;
		}
	}
	
	fclose(fp);
	
	return 0;
}
```

#### 3. 用读写接口实现 cat 命令

实现简单 `cat` 命令：
```c
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	if(argc != 2)
	{
		printf("argv error!\n");
		return 1;
	}
	
	FILE *fp = fopen(argv[1], "r");
	if(!fp){
		printf("fopen error!\n");
		return 2;
	}
	
	char buf[1024];
	while(1){
		int s = fread(buf, 1, sizeof(buf), fp);
		if(s > 0){
			buf[s] = 0;
			printf("%s", buf);
		}
		if(feof(fp)){
			break;
		}
	}
	
	fclose(fp);
	
	return 0;
}
```

```
./可执行程序 追加文件
```

---

# 三、系统文件 IO：从底层出发

虽然 C 库函数很好用，但在底层，真正和操作系统打交道的是系统调用。`man 2 open` 可以查看底层接口

## 3.1 `open` 函数解析

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

## 3.2 `write` 与 `read`

```c
#include <unistd.h>

ssize_t write(int fd, const void *buf, size_t count);
ssize_t read(int fd, void *buf, size_t count);
```

### 1. 实战：系统调用版写文件

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

### 2. 实战：系统调用版读文件

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

---

# 四、文件描述符 (File Descriptor)

在 C 语言层，我们通过 `FILE*` 操作文件；但在系统层，**文件描述符（fd）** 就是文件的唯一身份证

## 4.1 为什么新建文件的 fd 总是 3？

Linux 进程在启动时，默认会打开三个文件描述符：
*   **0**：标准输入（键盘）
*   **1**：标准输出（显示器）
*   **2**：标准错误（显示器）

当你调用 `open` 打开新文件时，分配规则是：**在文件描述符表中，找到当前没有被使用的、最小的一个下标**

## 4.2 进程与文件的关系（底层结构）

在操作系统的内核中：
1. 每个进程的 `task_struct`（PCB）里都有一个指针 `files`
2. 指向一张 `files_struct` 表
3. 这张表最重要的部分是一个**指针数组** `fd_array[]`
4. 数组的**下标**就是文件描述符（fd），数组的**内容**是指向改文件的 `struct file` 结构体的指针

![](图片/文件描述符.png)

## 4.3 文件描述符的分配规则

以下新文件输出 fd = 3
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
	int fd = open("myfile", O_RDONLY);
	if(fd < 0){
		perror("open");
		return 1;
	}
	printf("fd: %d\n", fd);
	
	close(fd);
	return 0;
}
```

以下新文件输出 fd = 0
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main()
{
	close(0);
	int fd = open("myfile", O_RDONLY);
	if(fd < 0){
		perror("open");
		return 1;
	}
	printf("fd: %d\n", fd);
	
	close(fd);
	return 0;
}
```

>文件描述符分配规则：
>在 files_struct 数组中，找到当前没有被使用的最小的下标，作为文件描述符

##  4.4 重定向的本质

**现象**：如果你先 `close(1)`（关闭标准输出），然后再 `open` 一个文件。这个新文件的 fd 会分配为 1
此时你调用 `printf("hello\n");`，内容不会打印到屏幕，而是写入了这个文件！这就是**输出重定向 (`>`)** 的底层原理

**原理**：`printf` 默认往标准输出（fd 1）写数据。因为 fd 1 的下标重新被指向了磁盘文件，数据自然就流向了文件

**代码实践**：
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

int main()
{
	close(1);
	int fd = open("myfile", O_WRONLY | O_CREAT, 00644);
	if(fd < 0)
	{
		perror("open");
		return 1;
	}
	printf("fd: %d\n", fd);
	fflush(stdout);
	
	close(fd);
	exit(0);
}
```

## 4.5 核心系统调用：`dup2`

系统提供了一个强大的函数来修改 fd 的指向：

```c
#include <unistd.h>
int dup2(int oldfd, int newfd);
```

它会把 `oldfd` 复制给 `newfd`。如果 `newfd` 已经被打开，它会自动关闭

**重定向实战代码**：
```c
#include <stdio.h>
#include <unistd.h>
#inlcude <fcntl.h>

int main()
{
	// 此时 fd 的值可能是 3
	int fd = open("./log.txt",O_CREAT | O_ROWR);
	if(fd < 0)
	{
		perror("open");
		return 1;
	}
	
	close(1);
	dup2(fd, 1); // 让 1 号描述符（标准输出）指向 log.txt
	for(;;)
	{
		char* buf[1024] = buffer{0};
		ssize_t read_size = read(0, buf, strlen(buf) - 1);
		if (read_size < 0)
		{
			perror("read");
			break;
		}
		printf("%s", buf); // 因为此时 1 号已经是 log.txt 了
		fflush(stdout);
	}
	return 0;
}
```

通过 `dup2`，我们就能在自己的 Mini-Shell 中轻松实现 `<`、`>`、`>>` 等重定向功能了

---

# 五、深入理解“一切皆文件”

在 Windows 中，文件就是文件，设备就是设备。但在 Linux 的世界里，**“一切皆文件”**

键盘、显示器、磁盘、网卡，甚至进程、管道、Socket 网络套接字，都被抽象成了文件。这意味着，开发者**仅需要使用一套 API（`open`, `read`, `write`, `close`），就可以调取 Linux 系统中绝大部分的资源**

## 5.1 为什么能这么抽象？底层源码揭秘

当我们在 Linux 中打开一个文件时，操作系统会为该文件创建一个 `struct file` 结构体（定义在 `include/linux/fs.h` 中）

在这个结构体里，有两个非常关键的成员：
```c
struct file {
    // ...
    struct inode *f_inode;              /* 指向文件的元数据（权限、大小、所在磁盘块） */
    const struct file_operations *f_op; /* 指向该文件的操作方法集！核心！ */
    atomic_long_t f_count;              /* 文件的引用计数，类似 shared_ptr */
    unsigned int f_flags;               /* 打开文件时传入的标志位 (O_RDONLY 等) */
    loff_t f_pos;                       /* 当前读写指针的位置 */
    // ...
};
```

**答案就在 `f_op` 指针里**。它指向一个 `file_operations` 结构体，这个结构体里面全是**函数指针**：

```c
struct file_operations {
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    int (*open) (struct inode *, struct file *);
    // ... 等等各种操作
};
```

![](图片/操作系统是如何管理硬件的%201.png)

## 5.2 VFS（虚拟文件系统）的巧妙设计

1. **对于普通磁盘文件**：`f_op` 里的 `read` 指针，会指向文件系统（如 ext4）的读函数，最终去读磁盘。
2. **对于键盘（字符设备）**：`f_op` 里的 `read` 指针，会指向键盘驱动程序的读函数，等待硬件中断获取按键。
3. **对于网卡**：`f_op` 里的 `write` 指针，会指向网卡驱动的发送函数。

**结论**：尽管底层硬件千差万别，操作方法完全不同。但 Linux 内核在中间加了一层 `struct file` 的接口（这就是**VFS**）。无论你操作什么，只要拿着文件描述符（fd），调用系统的 `read`，内核就会通过 `f_op` 自动分发到对应的驱动程序去执行。 

**这就是“一切皆文件”的底层实现（多态思想在 C 语言中的完美体现）**

---

# 六、缓冲区 (Buffer)：用户态与内核态的桥梁

## 6.1 什么是缓冲区？为什么要引入？

**缓冲区就是内存中的一块存储空间**，用来暂存输入或输出的数据。

**原因**：CPU 处理速度极快，而磁盘、打印机等外设速度极慢。如果没有缓冲区，CPU 每执行一次 `write` 就要死等磁盘写完，这叫**系统调用开销**（涉及用户态与内核态的切换、进程上下文的保存）

有了缓冲区，CPU 把数据丢进内存缓冲区就可以去干别的了，由操作系统或 C 库在背后统一、批量地刷入磁盘

## 6.2 标准 I/O 缓冲区的三种类型

C 标准库（`stdio.h`）为 `FILE*` 流提供了三种缓冲策略：
*   **全缓冲区（Full Buffered）**：填满整个缓冲区（通常 4096 字节）才进行真正的 I/O 操作。**对于普通磁盘文件的操作，默认是全缓冲**
*   **行缓冲区（Line Buffered）**：遇到换行符 `\n` 时，才进行真正的 I/O 操作。**对于标准输入/输出（显示器/键盘），默认是行缓冲**
*   **无缓冲区（Unbuffered）**：不缓存，直接调用系统调用写入。**标准错误流 `stderr` 默认是无缓冲的**，保证错误信息能立刻打印出来

> [!note] 什么时候会刷新缓冲区？
> 1. 缓冲区满了
> 2. 遇到 `\n`（行缓冲）
> 3. 调用 `fflush()` 强制刷新
> 4. 进程正常退出（调用 `exit()` 或 `main` 返回）时，会自动刷新并清空缓冲区

##  6.3 如何验证缓冲区类型

### 1. 重定向文件需要 fflush 刷新缓冲区

**代码示例**：
```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
	close(1);
	int fd = open("log.txt", O_WRONLY | O_CREAT | O_TRUNC, 0666);
	if (fd < 0)
	{
		perror("open");
		return 1;
	}
	
	printf("hello world: %d\n", fd);
	// fflush(stdout);  // 文件默认是全缓冲，需要用到 fflush 刷新缓冲区
	close(fd);
	
	return 0;
}
```

我们想使用重定向思维，让输入的数据写入 `log.txt` 文件中，但程序运行结束后并没有数据写入

这是由于将磁盘文件重定向 1 号描述符后，缓冲区的刷新方式变成了全缓冲。可以使用 `fflush` 强制刷新缓冲区

### 2. 用标准错误 stderr 验证

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
	close(2);
	int fd = open("log.txt", O_WRONLY | O_CREAT | O_TRUNC, 0666);
	if (fd < 0)
	{
		perror("open");
		return 1;
	}
	
	perror("hello world"); // 手动标准错误，看是否打印在文件中
	close(fd);
	
	return 0;
}
```

标准错误直接输出进 `log.txt`，说明标准错误不需要 `fflash` 刷新缓冲区

## 6.4 经典陷阱：`fork` 为什么会导致 `printf` 输出两次

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main()
{
	const char* msg1 = "hello printf\n";
	const char* msg2 = "hello fwrite\n";
	const char* msg3 = "hello write\n";
	
    printf("%s", msg1);            // C库函数，写入 stdout 缓冲区
    fwrite(msg2, 1, strlen(msg2), stdout); // C库函数，写入 stdout 缓冲区
    write(1, msg3, strlen(msg3));  // 系统调用，直接写入内核，无用户缓冲区
    
    fork(); // 创建子进程
	return 0;
}
```

- **现象**：程序正常运行（输出到显示器，行缓冲，遇 `\n` 瞬间刷新），输出 `printf`, `fwrite`, `write` 各一次，共 3 行
- **异常**：如果执行 `./a.out > file.txt`（重定向到磁盘文件）。输出变成了 5 行！`printf` 和 `fwrite` 都输出了两次，而 `write` 依然只有一次

**底层原理剖析**：

1. `printf` 和 `fwrite` 是 C 库函数，它们在用户空间维护了一块**用户态缓冲区**。而 `write` 是系统调用，没有用户态缓冲区，直接进内核
2. 发生重定向后，数据写入磁盘文件，缓冲方式**从 行缓冲 变成了 全缓冲**。此时 `printf` 的字符串虽然带 `\n`，但并不会立刻刷新，而是被暂存在用户态的缓冲区里
3. **在缓冲区未刷新时，执行了 `fork()`**。子进程发生**写时拷贝**，连带着把父进程用户态的缓冲区数据也复制了一份
4. 当父子进程分别退出时，它们各自刷新各自的缓冲区，导致文件里出现了两份相同的数据
5. 而 `write` 是直接写到内核页缓存的，没有用户态缓冲区的拷贝问题，所以只输出一次

>[!note] **总结**：`printf` / `fwrite` 有用户态缓冲区，`write` 没有。这就是为什么说库函数是对系统调用的二次封装，而缓冲区是 C 库自己加的

## 6.5 FILIE 结构体

在 `/usr/include/libio.h` 中，`FILE` 的真名叫 `_IO_FILE`。里面不仅有 `_fileno`（封装了系统调用的 fd），还有一对指针 `_IO_write_base` 和 `_IO_write_ptr`，分别指向缓冲区的起始位置和当前写入位置。

> **补充说明**：我们上面讨论的全部是**用户态缓冲区**。实际上，操作系统在底层（内核态）也有**内核缓冲区（Page Cache）**。库函数把数据刷新到内核缓冲区后，真正落盘由内核决定（比如调用 `fsync` 才会强行落盘）

如果感兴趣，可以看看 FILE 结构体：
`typedef struct _IO_FILE FILE; 在/usr/include/libio.h`
```c
/* 在/usr/include/libio.h */
struct _IO_FILE
{
    int _flags; /* High-order word is _IO_MAGIC; rest is flags.*/
#define _IO_file_flags _flags

    // 缓冲区相关
    /* The following pointers correspond to the C++ streambuf protocol. */
    /* Note: Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
    char *_IO_read_ptr;   /* Current read pointer */
    char *_IO_read_end;   /* End of get area. */
    char *_IO_read_base;  /* Start of putback+get area. */
    char *_IO_write_base; /* Start of put area. */
    char *_IO_write_ptr;  /* Current put pointer. */
    char *_IO_write_end;  /* End of put area. */
    char *_IO_buf_base;   /* Start of reserve area. */
    char *_IO_buf_end;    /* End of reserve area. */
    /* The following fields are used to support backing up and undo. */
    char *_IO_save_base;   /* Pointer to start of non-current get area. */
    char *_IO_backup_base; /* Pointer to first valid character of backup area*/
    char *_IO_save_end;    /* Pointer to end of non-current get area. */
    
    struct _IO_marker *_markers;
    
    struct _IO_FILE *_chain;
    
    int _fileno; // 封装的⽂件描述符
#if 0
int _blksize;
#else
    int _flags2;
#endif
    _IO_off_t _old_offset; /* This used to be _offset but it's too small. */
    
#define __HAVE_COLUMN      /* temporary */
    /* 1+column number of pbase(); 0 is unknown. */
    unsigned short _cur_column;
    signed char _vtable_offset;
    char _shortbuf[1];
    
    /* char* _save_gptr; char* _save_egptr; */
    
    _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```