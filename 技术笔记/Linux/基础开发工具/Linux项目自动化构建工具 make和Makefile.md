# # 自动化构建：Make 与 Makefile

如果一个项目有 100 个 `.c` 文件，每次改一个都要手敲 100 遍 `gcc` 吗？当然不！这就是 `make` 工具和 `Makefile` 文件的作用

## Makefile 的核心规则

```shell
# Makefile

# 目标文件:依赖文件
myproc: myproc.c           # 可以是多个依赖文件
	gcc -o myproc myproc.c # 第二行必须以tap建为开头，以什么方式编译
```
_(⚠️注意：第二行必须用 Tab 键缩进，不能用空格！)_

当你执行 `make` 命令时，它会寻找当前目录下的 `Makefile`，并执行第一条规则

## 为什么不能多次执行 make

执行一次 `make` 生成 `myproc` 后，如果没修改过 `myproc.c`，再次 `make` 会提示 已是最新

- **原理**：`make` 会比较 **目标文件** 和 **依赖文件** 的修改时间。如果 `myproc.c`（依赖）比 `myproc`（目标）老，说明源码没改过，`make` 就会跳过编译以节省时间

## 伪目标 `.PHONY` 与清理规则

通常我们需要一个清理命令（`make clean`），为了防止目录下正好有个文件叫 `clean` 导致规则冲突，我们使用 `.PHONY` 声明它为**伪目标**，让它无视时间对比，永远被执行

```bash
# Makefile

myproc:myproc.c
	gcc -o myproc myproc.c

.PHONY:clean # .PHONY:修饰符  修饰 clean
clean:       # 依赖为空，语法规则一样
	rm -f myproc
```

```bash
# 命令行中执行，清除文件
make clean # 出现 rm -f myproc (也就是我们在 Makefile 中写入的 clean)
```

## Makefile 定义变量

`Linux` 中的 `Makefile` 里定义变量与C语言中的宏定义很像，本质是替换。C语言的语法为 `#default`、`Linux` 中则为 `$()`
特殊语法：`$()` 替换内容、`$^` 表示依赖文件、`$@` 表示目标文件
```bash
# 定义的变量
BIN=proc.exe
CC=gcc
SRC=myproc.c
FLAGS=-o
RM=rm -f

# 替换$()
$(BIN):$(SRC)
$(CC) $(FLAGS) $@ $^ # $@ = BIN   $^ = SRC
.PHONY:
clean:
$(RM) $(BIN)

.PHONY:test
test:
@echo $(BIN)     # 输出 proc.exe
@echo $(CC)      # 输出 gcc
@echo $(SRC)     # 输出 myproc.c
@echo $(FLAGS)   # 输出 -o
@echo $(RM)      # 输出 rm -f
```


## Makefile 进阶：变量与自动推导

在大型工程中，我们使用变量和通配符来简化编写：

- `$@`：代表目标文件名（如 `myproc`）
- `$^`：代表所有依赖文件名（如 `myproc.c`）
- `$<`：代表第一个依赖文件名
- `$(wildcard *.c)`：获取当前目录下所有的 `.c` 文件

```shell
# Makefile

BIN = proc.exe
CC = gcc
SRC = $(wildcard *.c)
OBJ = $(SRC:.c=.o)

$(BIN): $(OBJ)
	@$(CC) -o $@ $^
%.o: %.c
	@$(CC) -c $<

.PHONY: clean
clean:
	rm -f $(OBJ) $(BIN)
```

## Makefile 扩展

### Makefile 顺序问题

我们把上面写的 `Makefile` 拆解一番
```bash
# 第一个命令(make)
myproc:myproc.c
	gcc -o myproc myproc.c

.PHONY:clean # 伪目标

# 第二个命令(make clean)
clean:
	rm -f myproc
```
因为 `Makefile` 顺序问题，在执行 `make` 命令时，最先执行 `gcc -o myproc myproc.c`，只有明确告知需要用 `clean` 时，执行 `make clean` 才会执行 `rm -f myproc`

如果像下列这样写：
```bash
# 第一个命令(make)
clean:
	rm -f myproc

.PHONY:clean # 伪目标

# 第二个命令(make myproc)
myproc:myproc.c
	gcc -o myproc myproc.c
```
会因为顺序问题，执行 `make` 时就变成 `rm -f myproc`，**但我们都会把创建文件放在第一个**


### 文件信息

我们用 `stat myproc.c`，能看到文件的基本信息，包括：**查看时间、修改时间、属性修改时间**
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz lesson9]$ stat myproc.c
File: ‘myproc.c’
Size: 79 Blocks: 8 IO Block: 4096 regular file
Device: fd01h/64769d Inode: 139526 Links: 1
Access: (0664/-rw-rw-r--) Uid: ( 1001/ zjh) Gid: ( 1001/ zjh)
Access: 2026-03-14 22:50:56.325289332 +0800 # 查看时间
Modify: 2026-03-14 22:50:56.324289316 +0800 # 修饰时间
Change: 2026-03-14 22:50:56.324289316 +0800 # 修改属性时间
Birth: -
```

| 文件信息 | Access            | Modify | Change                           |
| ---- | ----------------- | ------ | -------------------------------- |
| 修改   | cat（只对第一次 cat 有效） | vim    | vim、chmod（modify 更改，Change也跟着更改） |

### Makefile 深层理解（推导过程）

>机器只能识别 `.o` 文件，那为什么 `Makefile` 文件中，可以写成 `myproc:myproc.c` 呢？

那是因为 Linux 中优化了 `Makefile`，省略了许多步骤，完整的路线图如下：
```bash
#myproc:myproc.c
#	gcc -o myproc myproc.c

myproc:myproc.o
gcc myproc.o -o myproc
myproc.o:myproc.s
gcc -c myproc.s -o myproc.o
myproc.s:myproc.i
gcc -S myproc.i -o myproc.s
myproc.i:myproc.c
gcc -E myproc.c -o myproc.i

.PHONY:clean
clean:
rm -f *.i *.s *.o myproc
```
- `myproc:myproc.o` 这里的 `myproc.o` 不存在，把 `myproc:myproc.o` 入栈
- `myproc.o:myproc.s` 这里的 `myproc.s` 不存在，把 `myproc.o:myproc.s` 入栈
- `myproc.s:myproc.i` 这里的 `myproc.i` 不存在，把 `myproc.s:myproc.i` 入栈
- `myproc.i:myproc.c` 这里的 `myproc.c` 存在，依次出栈并执行
- 从此就完成了从上到下入栈，从下到上执行的 `Makefile` 推导规则


### 多文件 Makefile（手写链接）

多文件的 `Makefile` 写法 与  上述单文件的 `Makefile` 写法不同，如下：将多个文件编译成 `.o`，统一链接
```bash
BIN=proc.exe
CC=gcc
SRC=myproc.c
OBJ=myproc.o
LFLAGS=-o
FLAGS=-c
RM=rm -f

$(BIN):$(OBJ)
	$(CC) $(LFLAGS) $@ $^
%.o:%.c # 把当前路径下所有的 .o/.c 依次展开
	$(CC) $(LFLAGS) $< # 对源文件依次进行编译

.PHONY:clean
clean:
	$(RM) $(OBJ) $(BIN)
```

不想每次 `make` 和 `make clean` 打印命令，可以用 `@` 来注释掉
```bash
BIN=proc.exe
CC=gcc
SRC=myproc.c
OBJ=myproc.o
LFLAGS=-o
FLAGS=-c
RM=rm -f

$(BIN):$(OBJ)
	@(CC) $(LFLAGS) $@ $^
	@echo "linking ... $^ to $@"
%.o:%.c # 把当前路径下所有的 .o/.c 依次展开
	@$(CC) $(LFLAGS) $< # 对源文件依次进行编译
	@echo "compling ... $< to $@"

.PHONY:clean
clean:
	$(RM) $(OBJ) $(BIN)
```

### 多文件 Makefile（自动链接）

每次使用 `Makefile` 都要自己去用 `.o` 文件链接，如果 `.o` 文件特别多，这就会失去 `Makefile` 文件的便利性，因此就有**自动链接**的语法
```bash
BIN=proc.exe
CC=gcc
# SRC=$(shell ls *.c)
SRC=$(wildcard *.c)   # 显示当前目录下所有的.c文件名
OBJ=$(SRC:.c=.o)      # SRC内部的文件名.c -> 文件名.o
LFLAGS=-o
FLAGS=-c
RM=rm -f

$(BIN):$(OBJ)
	@(CC) $(LFLAGS) $@ $^
	@echo "linking ... $^ to $@"
%.o:%.c # 把当前路径下所有的 .o/.c 依次展开
	@$(CC) $(LFLAGS) $< # 对源文件依次进行编译
	@echo "compling ... $< to $@"

.PHONY:clean
clean:
	$(RM) $(OBJ) $(BIN)
	
.PHONY:test
test:
	@echo $(SRC)
	@echo $(OBJ)
```

**对Makefile（自动链接）进行测试**
```bash
# 在lesson9中输入
count=1; while [ $count -le 100 ]; do touch code${count}.c; let count++; done
```
可以看到创建了 100 个 `.c` 文件，此时使用 `make` 和 `make clean` 时，就可以看到创建 `.o` 和 `链接` 的输出