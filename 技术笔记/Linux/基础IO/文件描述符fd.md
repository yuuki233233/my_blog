# 文件描述符 (File Descriptor)

在 C 语言层，我们通过 `FILE*` 操作文件；但在系统层，**文件描述符（fd）** 就是文件的唯一身份证

## 为什么新建文件的 fd 总是 3？

Linux 进程在启动时，默认会打开三个文件描述符：
*   **0**：标准输入（键盘）
*   **1**：标准输出（显示器）
*   **2**：标准错误（显示器）

当你调用 `open` 打开新文件时，分配规则是：**在文件描述符表中，找到当前没有被使用的、最小的一个下标**

## 进程与文件的关系（底层结构）

在操作系统的内核中：
1. 每个进程的 `task_struct`（PCB）里都有一个指针 `files`
2. 指向一张 `files_struct` 表
3. 这张表最重要的部分是一个**指针数组** `fd_array[]`
4. 数组的**下标**就是文件描述符（fd），数组的**内容**是指向改文件的 `struct file` 结构体的指针

![](图片/文件描述符.png)

### 文件描述符的分配规则

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

##  重定向的本质

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

## 核心系统调用：`dup2`

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