## 文件描述符
>每个文件都有对应的文件描述符，语言层看文件的路径，而系统只看文件的描述符。
>问题：
>1. 当打开第一个文件时，为什么文件描述符 fd = 3
>2. 文件描述符 0 1 2 分别是什么
>3. 进程与文件有何种关系

#### 文件描述符 0 1 2
- linux 进程默认情况下会自动打开 3 个文件描述符，分别是标准输入0、标准输出1、标准错误2
- 0 1 2 对应的物理设备：键盘，显示器，显示器
- ![](图片/文件描述符.png)
>- 文件描述符从 0 开始的整数
>- 打开个文件，就把文件的指针放入数组的最小空位
>- task_struct进程 里有 files_struct文件指针数组，数组里存打开的文件
>- 数组下标对应文件描述符，可以用文件描述符找到对应的文件

#### 文件描述符的分配规则

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