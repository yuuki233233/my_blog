>**作者**：yuuki233233
>**目标**：德国 CS 本科 + 特斯拉软件工程师
>**适用人群**：大一/自学者，想快速上手 Linux 命令行 + 搞懂权限本质
>
>这篇博客主要讲解了软件包管理器、编译器、vim 编制器、自动化构建工具、gdb 调试的作用，这是一份针对 Linux 的实用开发工具，是学习 Linux 的重要路程
>
>欢迎点赞、收藏、评论，一起卷 Linux！

# 1. 软件包管理器

## 1.1 Linux 软件安装三种方式

| 方式                      | 优点                 | 缺点                | 推荐场景        |
| ----------------------- | ------------------ | ----------------- | ----------- |
| 包管理器（yum/dnf/apt）       | 自动解决依赖、版本兼容、卸载干净   | 软件版本可能较旧、需要配置镜像加速 | 服务器、生产环境首选  |
| 源码编译安装                  | 最新版本、可自定义配置、优化编译参数 | 依赖手动解决、编译耗时、卸载麻烦  | 需要最新版或特殊优化时 |
| 二进制包（.deb/.rpm/.tar.gz） | 安装快、免编译            | 依赖问题、路径不标准、可能污染系统 | 临时用或官方不提供包时 |

- 二进制包本质就是把编译好的可执行文件、库、配置文件、手册页等，按标准路径 `（/usr/bin、/usr/lib、/etc、/usr/share/man）` 直接解压/拷贝过去，所以叫`“安装 = 拷贝资源”`
- **包管理器是服务器/日常使用首选**，因为它把“下载 → 依赖检查 → 安装 → 注册服务 → 更新源”全部自动化了。 在 Ubuntu/Debian 系上，最常用的是 **apt**（底层是 dpkg）。

## 1.2 apt 具体操作

### 1.2.1 apt 常用命令速查

| 命令                          | 作用             | 示例                              |
| --------------------------- | -------------- | ------------------------------- |
| `sudo apt update`           | 更新软件源索引（必须先执行） | 每次换源或想获取最新包列表时运行                |
| `sudo apt upgrade`          | 升级所有已安装软件      | 保持系统最新，推荐定期运行                   |
| `sudo apt install 软件名`      | 安装软件（自动解决依赖）   | `sudo apt install vim git curl` |
| `sudo apt remove 软件名`       | 卸载软件（保留配置文件）   | `sudo apt remove nginx`         |
| `sudo apt purge 软件名`        | 彻底卸载（连配置文件一起删） | `sudo apt purge nginx`          |
| `sudo apt autoremove`       | 自动清理不再需要的依赖包   | 卸载软件后运行，释放空间                    |
| `sudo apt search 关键词`       | 搜索软件包          | `sudo apt search python3-pip`   |
| `sudo apt list --installed` | 列出已安装软件        | 查看系统中装了哪些包                      |

### 1.2.2 apt 源配置文件（sources.list）
Ubuntu 的软件源配置文件在：
```text
/etc/apt/sources.list
/etc/spt/sources.list.d/*.list
```


**换国内镜像加速**（推荐，国外源在中国慢得离谱）：
1. 备份原文件
   ```bash
   sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
   ```

2. 用国内镜像替换（以阿里云为例，推荐速度最稳）
  
```Bash
sudo sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/security.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list
```

3. 更新索引

```Bash
sudo apt update
```

其他常用镜像：

- 清华：`mirrors.tuna.tsinghua.edu.cn`
- 中科大：`mirrors.ustc.edu.cn`
- 网易：`mirrors.163.com`

**小技巧**：换源后如果报错，先运行 `sudo apt update --fix-missing` 修复。

# 2. 编译器 gcc 和 g++
## 2.1 编译器处理代码过程
**gcc编译选项**
```bash
gcc [选项] 要编译的文件 [选项] [目标文件]
```

**编译器处理过程**
1. 预处理（进行宏替换/去注释/条件编译/头文件展开等）
2. 编译（生成汇编）
3. 汇编（生成机器可识别代码）
4. 链接（生成可执行文件或库文件）

### 2.1.1 预处理（宏替换）
- 运行：`gcc -E hello.c -o hello.i`
- 预处理功能主要包括宏定义，文件包括，条件编译，去注释等
- 预处理指令是以 `#` 号开头的代码行
- 选项 `-E` 的作用：让 gcc 在预处理结束后停止编译
- 选项 `-o` 的作用：写入到目标文件，`.i` 文件为预处理后的 C 原始程序

### 2.1.2 编译（生成汇编）
- 运行：`gcc –S hello.i –o hello.s`
- gcc 首先检查代码的规范性，无误后 gcc 把代码**翻译成**汇编语言
- 选项 `-S` 的作用：进行查看，只进行编译而不进行汇编，**生成**汇编代码

### 2.1.3 汇编（生成机器可识别代码）
- 运行：`gcc –c hello.s –o hello.o`
- 把编译生成的 `.s` 文件**转成**目标文件
- 选项 `-c` 的作用：看到汇编代码已经**转化**为 `.o` 的二进制目标代码

### 2.1.4 链接（生成可执行文件或库文件）
- 运行：`gcc hello.o –o hello`
- 成功编译，进入链接阶段

**程序在翻译时，喜欢做的**：先编译成 `.o`，再链接
- 编译器在编译时，不仅仅要形成可执行文件，还要形成库，如果形成库的话，就不需要编译成可执行程序，必须形成 `-o`，最后进行操作。
- 目前使用的 VS 编译时，就只形成一个可执行程序，如果文件过多就会形成多个可执行程序，损耗效率。

**多个文件：** 将全部 `.c` 打包成 `.o`，再进行链接形成可执行文件
```bash
gcc code.o code1.o code2.o -o code
```

## 2.2 动态链接和静态链接
---
### 2.1 动态链接
动态链接是在编译还没加载时就链接好了，动态链接只是把我们要访问的库资源的地址写到我的程序当中，当我加载之后动态找到对应的库就执行
 
```C
/* 库 */                   /* 代码 */

// 地址：libc.so:地址
int printf(...)              for()
{ ...; }                    { // 链接动态库，把printf替换成libc.so:地址
		                        printf("hello world"); 
	                        }
```

**动态链接的好处：**
- 把程序分成各个独立部分，运行时链接在一起（不像静态链接，把所有程序模块都链接成一个单独的可执行文件）

### 2.2 静态链接
把程序中使用的库拷贝给我自己，静态库只有在链接的时候有用，一旦形成可执行程序，静态库可以不再需要

```C
/* 库 */                   /* 代码 */

// 地址：libc.so:地址
int printf(...)              for()
{ ...; }                    { // 拷贝一份libc.so:地址给printf用
		                        printf("hello world"); 
	                        }
```

**静态链接的好坏：**
- 运行速度快：可执行程序中具备所有执行程序所需要的任何东西
- 浪费空间：每个可执行程序中对所有需要的目标文件都要有一份副本，所有多个程序会依赖同个文件，同一个目标文件都在内存存在多个副本
- 更新困难：每改库函数是，需要重新进行编译链接形成可执行程序。
---
## 2.3 静态库和动态库

- **静态库**（libc.a)：编译链接是，把库文件的代码全部加入到可执行文件中
- **结果**：生成文件较大，运行时就不需要库文件（后缀名一般为 `.a`）

- **动态库**（libc.so）：与静态库相反
- **结果**：编译链接时而是在程序执行时由运行时链接文件加载库，节省开销（后缀名为 `.so`）

我们自己的程序 + 动态库内部实现的方法 -> 形成可执行程序（让我们自己的程序能在库中找到方法，需跳转到库中执行，完了再返回）

**动静态库对比**
1. 动态库形成的可执行程序体积一定很小
2. 可执行程序对静态库的依赖度小，动态库不能缺失
3. 程序运行，需要加载内存，静态链接的会在内存中出现大量重复代码
4. 动态链接用的是动态库的地址，比较节省内存和磁盘资源

>gcc 在编译时默认使用动态库，完成链接后就可以生成可执行文件
>动态库：linux(.so) windows(.dll)
>静态库：linux(.a) windows(.lib)

# 3. vim 使用
vim 编辑器分为五种模式：命令模式（command mode）、插入模式（insert mode）、底行模式（last line mode）、替换模式、注释模式

>命令模式可以转成其他模式，其他模式只能转回命令模式

## 3.1 命令模式（command mode）

| 命令                                 | 模式     |
| ---------------------------------- | ------ |
| `i（insert）/ o（往下空新起一行）/ a（往后移一字符）` | `插入模式` |
| `:`                                | `末行模式` |
| `R`                                | `替换模式` |
| `ctrl + v`                         | `注释模式` |

### 3.1.1 移动光标（在命令模式下）

| 目的     | 指令                          |
| ------ | --------------------------- |
| 左下上右   | `h(左) j(下) k(上) l(右)`       |
| 顶部     | `gg`                        |
| 底部     | `G`                         |
| 指定行    | `[行数] + G`                  |
| 跳跃单个单词 | `w(右) b(左)`                 |
| 跳跃多个单词 | `[跳跃单词数] + w 、[跳跃单词数] + b ` |
| 行首     | `^`                         |
| 行尾     | `$`                         |

### 3.1.2 其他操作

| 目的           | 指令                |          |
| ------------ | ----------------- | -------- |
| 复制当 前行/多行    | `yy` / `[数] + yy` |          |
| 光标位置粘贴 一次/多次 | `p` / `[数] + p`   |          |
| 向前撤销         | `u`               |          |
| 向后撤销         | `ctrl + r`        |          |
| 截切 一行/多行     | `dd` / `[数] + dd` |          |
| 右删除字符（可复制）   | `x` / `[数] + x`   |          |
| 左删除字符（可复制）   | `X` / `[数] + X`   |          |
| 替换光标所在字符     | `r` / `[数] + r`   |          |
| 大小写切换        | `~`               |          |
| 选中单词         | `#`               | `n`：逆向查找 |

## 3.2 插入模式（insert mode）

- 可进行**文字输入**
- 按 `Esc` 回到命令模式

## 3.3 底行模式（last line mode）

| 命令                   | 用途          |
| -------------------- | ----------- |
| `w`                  | 保存          |
| `q`                  | 退出          |
| `!q`                 | 强制退出        |
| `set nu`             | 设置行号        |
| `! [命令]`             | 在`vim`中使用命令 |
| `%s /dst/srt/`       | 批量化替换       |
| `vs 新文件`、`ctrl + ww` | 分屏、切换文件     |
| `/单词`                | 查找单词        |
| `vim 文件 + 行号`        | 让光标指到对应位置   |
| `!v`                 | 执行上一个底行指令   |
| `Esc`                | 回到命令模式      |

## 3.4 注释模式

| 步骤  | 批量化注释                  | 区域删除          | 区域编写           |
| --- | ---------------------- | ------------- | -------------- |
| 1   | `h j k l`区域选择，可与其他命令结合 | `h j k l`区域选择 | `h j k l`区域选择  |
| 2   | `I（大写）` 回到插入模式         | `d` 删除选择区域    | `I（大写）` 回到插入模式 |
| 3   | 输入 `//`                |               | 输入想要加的字符       |
| 4   | 按 `Esc`，批量注释成功         |               | 按 `Esc`，区域编写成功 |
按 `Esc` 回到命令模式

## 3.5 替换模式
- 按 `Esc` 回到命令模式

# 4. Linux项目自动化构建工具 make 和 Makefile

## 4.1 为什么有 make 和 makefile

- 一个工程的源文件不计其数，按类型、功能、模块分别放在若干个目录中，`makefile` 定义了一个系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至进行更复杂的功能操作
- **makefile 好处**：自动化编译（一旦写好，只需一个 `make` 命令让整个工程完全自动编译）
- **make（命令）**：解释 `makefile` 中指令的命令工具

## 4.2 make 和 makefile 的基础使用
可写成 `makefile` 或 `Makefile`

### 4.2.1 创建文件
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

### 4.2.2 删除文件

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

### 4.2.3 Makefile 顺序问题

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

### 4.2.4 文件属性 及 时间问题

在执行多个 `make` 时，只有第一次是通过的，其余都报错，那是因为两个文件 `myproc.c` `myproc` 的属性问题而导致的

#### （1）文件信息

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
#### （2）Modify 时间问题

- 不能  `make` 多次，与 `Modify` 的修改时间息息相关
- 想用多次 `make` 需要更改 `Modify` 的修改时间

#### （3）为什么不被执行多次 make

1. 默认老代码不做重新编译（当有1000份文件时，只修改了10份文件，就只 `make` 修改过的文件，若` make 1000份文件`，对性能会有消耗）
2. 怎么不做重新编译的？（每个文件都有不同的修改时间(Modify)，以 `myproc` 和 `myproc.c` 为例：）
	- 当 `myproc.c` 时间比 `myproc` 晚时，`make` 时就不会执行，因为当成 `myproc.c` 是老文件，没有被修改
	- 要是 `myproc.c` 被修改时，`myproc.c` 时间比 `myproc` 早，`make` 时就会执行，因为当成 `myproc.c` 是新文件，已被修改

#### （4）.PHONY 伪目标使用

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

>结论：
>	.PHONY:让 make 忽略源文件和可执行目标文件的 M 时间对比
### 4.2.5 Makefile 深层理解（推导过程）

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

### 4.2.6 Makefile 定义变量

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

### 4.2.7 多文件 Makefile（手写链接）

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

### 4.2.8 多文件 Makefile（自动链接）

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

# 5. 调试器 gdb 和 cgdb

## 5.1 gdb/cgdb 使用前提
- gdb 与 cgdb 最大的区别是：cgdb 有动态过程，而 gdb 没有
- 程序有两种模式：`debug` 和 `release` 模式，`Linux gcc/g++` 出来的二进制程序默认是 `release`
- 要使用 gdb/cgdb 调试，必须在源代码生成二进制程序时加上 `-g` 选项
```bash
gcc code.c -o code -g    # debug模式

cgdb code    # 一定是要可执行文件，不是 code.c
```

## 5.2 gdb/cgdb 常见使用命令

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
