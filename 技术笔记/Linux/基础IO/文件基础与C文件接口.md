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