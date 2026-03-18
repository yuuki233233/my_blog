## 学习 gdb/cgdb 前的代码样例
```cpp
// mycmd.c
#include <stdio.h>

int Sum(int s, int e)
{
	int result = 0;
	for(int i = s; i <= e; i++)
	{
		result += i;
	}
	
	return result;
}

int main()
{
	int start = 1;
	int end = 100;
	printf("I will begin\n");
	int n = Sum(start, end);
	printf("running done, result is: [%d-%d]=%d\n", start, end, n);
	
	return 0;
}
```

## gdb/cgdb 使用前提
- gdb 与 cgdb 最大的区别是：cgdb 有动态过程，而 gdb 没有
- 程序有两种模式：`debug` 和 `release` 模式，`Linux gcc/g++` 出来的二进制程序默认是 `release`
- 要使用 gdb/cgdb 调试，必须在源代码生成二进制程序时加上 `-g` 选项
```bash
gcc code.c -o code -g    # debug模式

cgdb code    # 一定是要可执行文件，不是 code.c
```

## gdb/cgdb 常见使用命令

- 开始：`gdb binFile`
- 退出：`ctrl + d` 或 `quit` 调试命令

| 命令  | 作用  | 样例  |
| --- | --- | --- |
|     |     |     |
|     |     |     |
|     |     |     |
|     |     |     |
|     |     |     |
|     |     |     |
|     |     |     |
