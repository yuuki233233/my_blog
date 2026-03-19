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

| 命令                        | 作用                      | 样例                    |
| ------------------------- | ----------------------- | --------------------- |
| `list/l`                  | 显示源代码，从上次位置开始，每次列出10行   | `l 10`                |
| `list/l 函数名`              | 列出指定函数的源代码              | `l main`              |
| `list/l 文件名:行号`           | 列出指定文件的源代码              | `l mycmd.c:1`         |
| `r/run`                   | 从程序开始连续执行               | `r`                   |
| `n/next`                  | 单步执行，不进入函数内部（VS F10）    | `n`                   |
| `s/step`                  | 单步执行，进入函数内部（VS F11）     | `s`                   |
| `break/b 文件名:行号`          | 在指定行号设置断点               | `b 10`/`b test.c:10`  |
| `break/b 函数名`             | 在函数开头设置断点               | `b main`              |
| `info break/b`            | 查看当前所有断点的信息             | `info b`              |
| `finish`                  | 执行到当前函数返回，然后停止          | `finish`              |
| `print/p 表达式`             | 打印表达式的值                 | `p start+end`         |
| `p 变量`                    | 打印指定变量的值                | `p x`                 |
| `set var 变量=值`            | 修改变量的值                  | `set var i=10`        |
| `continue/c`              | 从当前位置开始连续执行程序           | `c`                   |
| `delete/d breakpoints`    | 删除所有断点                  | `d breakpoints`       |
| `delete/d n`              | 删除序号为n的断点               | `d 1`                 |
| `disable breakpoints`     | 禁用所有断点                  | `disable breakpoints` |
| `disable n`               | 禁用序号为n的断点               | `disable 1`           |
| `enable breakpoints`      | 启用所有断点                  | `enable breakpoints`  |
| `enable n`                | 启用序号为n的断点               | `enable 1`            |
| `info/i breakpoints`      | 查看当前设置的断点列表             | `info b`              |
| `display 变量名`             | 跟踪显示指定变量的值              | `display x`           |
| `undisplay 编号`            | 取消对指定编号的变量的跟踪显示         | `undisplay 1`         |
| `backtrace/bt`            | 查看当前执行栈的各级后汉书调用及参数      | `bt`                  |
| `until 行号`                | 执行到指定行号                 | `until 20`            |
| `info/i locals`           | 查看当前栈帧的局部变量值            | `info locals`         |
| `quit`                    | 退出GDB调试器                | `q`                   |
| `watch 变量`                | 当跟踪值变化时，GDB辉暂停程序的执行     | `watch x`             |
| `b 文件名/行号/函数名 if i == 30` | 给断点加上条件，条件满足时执行断点       | `b 10 if i == 20`     |
| `condition 序号 i == 30`    | 给存在的断点加条件               | `condition x i == 30` |
| `按Esc进入代码屏`/`按i退出`        | 进入代码屏，可以用滚轮滑动cgdb上显示的代码 | `按Esc进入代码屏`/`按i退出`    |
