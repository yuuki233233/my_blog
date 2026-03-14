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
会因为顺序问题，执行 `make` 时就变成 `rm -f myproc`

## 文件属性 及 时间问题

在执行多个 `make` 时，只有第一次是通过的，其余都报错，那是因为两个文件 `myproc.c` `myproc` 的属性问题而导致的

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
### 改写时间问题