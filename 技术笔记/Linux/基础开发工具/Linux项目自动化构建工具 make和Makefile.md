## make 和 makefile 是啥
make 是一个命令
makefile 是一个文件

## make 和 makefile 的基础使用

### 创建文件
先创建一个目录，在目录里分别创建 `Makefile` 与 `myproc.c` 2个文件
```bash
mkdir lesson9
cd lesson9

# lesson9
touch Makefile myproc.c
```
`myproc.c` 里只有输出 `hello world` 的代码

进入 `Makefile` 中，往里面写入下列命令
```bash
# Makefile

# 目标文件:依赖文件   myproc.c (怎么形成)-> myproc
myproc:myproc.c            # 可以是多个依赖文件
	gcc -o myproc myproc.c # 第二行必须以tap建为开头，以什么方式编译
```

输入 `make` 指令，自动输出一个 `myproc` 文件
```bash
# lesson9

make # 出现 gcc -o myproc myproc.c (也就是我们在 Makefile 中写入的)
```
此时出现一个 `myproc` 文件，`./myproc` 输出 `hello world`

### 删除文件

上面执行了创建文件的命令，那也可以执行删除的命令，进入 `Makefile`
```bash
# Makefile

myproc:myproc.c            # 可以是多个依赖文件
	gcc -o myproc myproc.c # 第二行必须以tap建为开头，以什么方式编译
.PHONY:clean # .PHONY:修饰符  修饰 clean
clean:       # 依赖为空，语法规则一样
	rm -f myproc
```

退出  `Makefile`
```bash
make clean # 出现 rm -f myproc (也就是我们在 Makefile 中写入的 clean)
```
此时 `myproc` 文件，已被删除

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

### 文件属性 及 时间问题

在执行多个 `make` 时，只有第一次是通过的，其余都报错，那是因为两个文件 `myproc.c` `myproc` 的属性问题而导致的

#### 文件信息

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
#### Modify 时间问题

- 不能  `make` 多次，与 `Modify` 的修改时间息息相关
- 想用多次 `make` 需要更改 `Modify` 的修改时间

#### 为什么不被执行多次 make

1. 默认老代码不做重新编译（当有1000份文件时，只修改了10份文件，就只 `make` 修改过的文件，若` make 1000份文件`，对性能会有消耗）
2. 怎么不做重新编译的？（每个文件都有不同的修改时间(Modify)，以 `myproc` 和 `myproc.c` 为例：）
	- 当 `myproc.c` 时间比 `myproc` 晚时，`make` 时就不会执行，因为当成 `myproc.c` 是老文件，没有被修改
	- 要是 `myproc.c` 被修改时，`myproc.c` 时间比 `myproc` 早，`make` 时就会执行，因为当成 `myproc.c` 是新文件，已被修改

#### .PHONY 伪目标使用

参考以下 `Makefile` 文件
```bash
.PHONY:clean
clean:
	rm -f myproc

.PHONY:clean
myproc:myproc.c
	gcc -o myproc myproc.c
```
会发现可以执行多次 `make`，是因为 `.PHONY` 类似一张通行证，可以一直反复往来一个地方，所以可以无视 `myproc` 文件，执行多次 `make`

为什么给**删除**通行：主要是因为项目清理时，要保证每一次清理时是干净的，就不会出现奇奇怪怪的错误

### Makefile 深层理解

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

### Makefile 定义变量

`Linux` 中的 `Makefile` 里定义变量与C语言中的宏定义很像，本质是替换。C语言的语法为 `#default`、`Linux` 中则为 `$()`
特殊语法：`$()` 替换内容、`$^` 表示源文件、`$@` 表示依赖文件
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

### 多文件 Makefile

多文件的 `Makefile` 写法 与  上述单文件的 `Makefile` 写法不同，如下
```bash
BIN=proc.exe
CC=gcc
SRC=myproc.c
OBJ=myproc.o
LFLAGS=-o
FLAGS=-c
RM=rm -f

$(BIN) : $(OBJ)
	@(CC) $(LFLAGS) $@ $^
	@echo "linking ... $^ to $@"
```