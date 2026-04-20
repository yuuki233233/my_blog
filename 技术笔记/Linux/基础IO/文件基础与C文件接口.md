## 理解“文件”

### 狭义理解

- 文件存在磁盘里，磁盘是个外设（即是输出设备也是输入设备）
- 磁盘是永久性存储介质，因此文件在键盘上的存储是永久性的
- 磁盘上的文件，本质是对文件的所有操作，都是对外设的输入和输出 IO

### 广义理解

- Linux 下一切皆文件（键盘、显示器、网卡、磁盘...）

### 文件操作的归类认识

- 文件 = 属性（元数据） + 内容
- 对于 0KB 的空文件时占用磁盘空间的
- 所有文件操作本质是对文件内容和属性的操作

### 系统角度

- 对文件的操作本质是进程对文件的操作
- 磁盘的管理者是操作系统
- 文件读写是通过文件相关的系统调用接口来实现（C/C++库函数也是调用系统接口实现文件读写）

## C文件接口

### 打开文件

```c
NAME
       fopen, fdopen, freopen - stream open functions

SYNOPSIS
       #include <stdio.h>

       FILE *fopen(const char *path, const char *mode);
```

```c
#include <stdio.h>

int main()
{
	FILE *fp = fopen("myfile", "w");
	if(!fp)
	{
		printf("fopen error!\n");
	}
	while(1);
	fclose(fp); // 关闭文件，但也不影响
	
	return 0;
}
```

此时创建 myfile 文件，没有写入任何数据

### 写文件

```c
NAME
       fread, fwrite - binary stream input/output

SYNOPSIS
       #include <stdio.h>

       size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

       size_t fwrite(const void *ptr, size_t size, size_t nmemb,
                     FILE *stream);

```

```c
#include <stdio.h>
#include <string.h>

int main()
{
	FILE *fp = fopen("myfile", "w");
	if(!fp)
	{
		printf("fopen error!\n");
	}
	
	const char *msg = "hello world\n";
	int count = 5;
	while(count--)
	{
		fwrite(msg, strlen(msg), 1, fp);
	}
	
	fclose(fp);
	
	return 0;
}
```

此时创建 myfile 文件，可以看到里面写入了 5行 hello world

### 读文件

```c
NAME
       fread, fwrite - binary stream input/output

SYNOPSIS
       #include <stdio.h>

       size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

       size_t fwrite(const void *ptr, size_t size, size_t nmemb,
                     FILE *stream);
```

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

### 输出信息到显示器的三种方法

```c
#include <stdio.h>
#include <string.h>

int main()
{
	const char *msg = "hello world!\n";
	fwrite(msg, strlen(msg), 1, stdout);
	
	printf("hello world!\n");
	
	fprintf(stdout, "hello world!\n");
	
	return 0;
}
```

### stdin stdout stderr

- C默认打开三个输入输出流，分别是 stdin，stdout，stderr
- 这三个流的类型都是 `FILE*`，`fopen` 返回值类型，文件指针
```c
#include <stdio.h>

extern FILE *stdin;
extern FILE *stdout;
extern FILE *stderr;
```

### 打开文件的方式

```c
DESCRIPTION
       The fopen() function opens the file whose name is the string pointed to by path and associates a stream with it.

       The argument mode points to a string beginning with one of the following sequences (possibly followed by additional characters, as described below):

       r      Open text file for reading.  The stream is positioned at the beginning of the file.

       r+     Open for reading and writing.  The stream is positioned at the beginning of the file.

       w      Truncate file to zero length or create text file for writing.  The stream is positioned at the beginning of the file.

       w+     Open for reading and writing.  The file is created if it does not exist, otherwise it is truncated.  The stream is positioned at the beginning of the file.

       a      Open for appending (writing at end of file).  The file is created if it does not exist.  The stream is positioned at the end of the file.

       a+     Open  for reading and appending (writing at end of file).  The file is created if it does not exist.  The initial file position for reading is at the beginning of the file, but output is always appended to the
              end of the file.
```