# 调试器：GDB 与 CGDB

写代码哪有不写出 Bug 的？`gdb` 是 Linux 下的命令行调试神器（`cgdb` 是它的可视化分屏升级版）

## 调试前提

默认 `gcc` 编译出的是 `release` 版本（无调试信息）。必须加 `-g` 选项生成 `debug` 版本：

```bash
gcc mycmd.c -o mycmd -g
cgdb mycmd   # 启动 cgdb 调试
```

## gdb/cgdb 常见使用命令

| 命令                        | 作用                            | 样例                    |
| ------------------------- | ----------------------------- | --------------------- |
| `list/l`                  | 显示源代码，从上次位置开始，每次列出10行         | `l 10`                |
| `list/l 函数名`              | 列出指定函数的源代码                    | `l main`              |
| `list/l 文件名:行号`           | 列出指定文件的源代码                    | `l mycmd.c:1`         |
| `r/run`                   | 从程序开始连续执行                     | `r`                   |
| `break/b 文件名:行号`          | 设置断点                          | `b 10`/`b test.c:10`  |
| `break/b 函数名`             | 函数开头设置断点                      | `b main`              |
| `info break/b`            | 查看当前所有断点的信息                   | `info b`              |
| `n/next`                  | 单步执行（不进入函数内部，相当于 VS 的 F10）    | `n`                   |
| `s/step`                  | 单步执行（**进入**函数内部，相当于 VS 的 F11） | `s`                   |
| `print/p 表达式`             | 打印变量的当前值                      | `p start+end`         |
| `continue/c`              | 继续运行到下一个断点                    | `c`                   |
| `finish`                  | 运行完当前函数并返回                    | `finish`              |
| `quit`                    | 退出调试                          | `q`                   |
| `p 变量`                    | 打印指定变量的值                      | `p x`                 |
| `set var 变量=值`            | 修改变量的值                        | `set var i=10`        |
| `delete/d breakpoints`    | 删除所有断点                        | `d breakpoints`       |
| `delete/d n`              | 删除序号为n的断点                     | `d 1`                 |
| `disable breakpoints`     | 禁用所有断点                        | `disable breakpoints` |
| `disable n`               | 禁用序号为n的断点                     | `disable 1`           |
| `enable breakpoints`      | 启用所有断点                        | `enable breakpoints`  |
| `enable n`                | 启用序号为n的断点                     | `enable 1`            |
| `info/i breakpoints`      | 查看当前设置的断点列表                   | `info b`              |
| `display 变量名`             | 跟踪显示指定变量的值                    | `display x`           |
| `undisplay 编号`            | 取消对指定编号的变量的跟踪显示               | `undisplay 1`         |
| `backtrace/bt`            | 查看当前执行栈的各级后汉书调用及参数            | `bt`                  |
| `until 行号`                | 执行到指定行号                       | `until 20`            |
| `info/i locals`           | 查看当前栈帧的局部变量值                  | `info locals`         |
| `watch 变量`                | 当跟踪值变化时，GDB辉暂停程序的执行           | `watch x`             |
| `b 文件名/行号/函数名 if i == 30` | 给断点加上条件，条件满足时执行断点             | `b 10 if i == 20`     |
| `condition 序号 i == 30`    | 给存在的断点加条件                     | `condition x i == 30` |
| `按Esc进入代码屏`/`按i退出`        | 进入代码屏，可以用滚轮滑动cgdb上显示的代码       | `按Esc进入代码屏`/`按i退出`    |

## 高级技巧：Watch 与 条件断点

- **`watch 变量`**：监控变量。当该变量的值发生改变时，程序会自动暂停。非常适合抓“变量莫名其妙被改变”的 Bug。
- **条件断点 `b [行号] if [条件]`**：比如 `b 9 if i == 30`。在循环了 1000 次的 `for` 循环里，你可以直接让程序在第 30 次循环时停下来，不用手动按 30 次 `continue`。
- **`set var 变量=值`**：在调试中动态修改变量的值，用来验证 Bug 是否由该变量引起。