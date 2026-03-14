## make 和 makefile 是啥
make 是一个命令
makefile 是一个文件

## make 和 makefile 的基础使用

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
myproc:myproc.c/.cpp       # 可以是多个依赖文件
	gcc -o myproc myproc.c # 第二行必须以tap建为开头，以什么方式编译
```

输入 `make` 指令，自动输出一个 `myproc` 文件
```bash
# lesson9

make # 出现 gcc -o myproc myproc.c (也就是我们在 Makefile 中写入的)
```
此时出现一个 `myproc` 文件，`./myproc` 输出 `hello world`

