# ls 指令
**语法：**  `ls [选项] [目录/文件]`
**功能：** 对于目录，该命令列出该目录下的所有子目录与文件。对于文件，将列出文件名以及其他信息。
**常用选项：**
>- -a 列出目录下的所有文件，包括以 `.`开头的隐藏文件
>- -l 列出文件的向下信息
>- -R 列出所有子目录下的文件。(递归)
>- -d 将目录像文件一样显示，而不是显示其下的文件。如：ls -d [指定目录]
>- -i 输出文件的`i`节点的索引信息。如：ls -ai [指定文件]
>- -k 以`k`字节的形式表示文件的大小。如：ls -alk [指定文件]
>- -n 用数字的UID，GID代替名称。(介绍UID，GID)
>- -f 在每个文件名后附上字符以说明该文件的类型，`*`表示可执行的普通文件；`/`表示目录；`@`表示符号链接；`|`表示`FIFOs`；`=`表示套接字。(目录类型识别)
>- -r 对目录反向排序
>- -t 以时间排序
>- -s 在`l`文件名后输出该文件的大小。(大小排序，如何找到目录下的最大的文件)
>- -1 一行只输出一个文件

# grep指令

# top
# zip/unzip指令(需安装)
## 一、安装zip/unzip指令
在使用zip和unzip命令时，要在对应的服务器进行安装
**Debian/Ubantu系列**
```Bash
apt update && apt install zip unzip -y
```
**unzip**也一同安装，解压zip文件会用到

**CentOS/RHEL系列**
```Bash
yum install zip unzip -y
```

# rz和sz命令(需安装)
## 一、安装sz/rz指令
rz、sz时用来在本地和服务器之间传文件的工具
**Debian/Ubantu系列**
```bash
sudo apt update && sudo apt install lrzsz -y
```
**CentOS/RHEL系列**
```bash
yum install -y lrzsz
```
## 二、sz/rz远程传输
**核心格式：** `sz 压缩包.zip`
**1.基础用法**
- Linux中传到Windows：`sz test.zip`
- Windows传到Linux：`rz`
- Linux传到Linux：`scp 压缩文件.zip 用户名@公网ip：目标机器指定路径`
# tar指令(Linux自带)
**核心格式：** `tar [参数] 文件/文件夹 ...`
**1.常实用参数**：
- `-c`：创建压缩包
- `-z`：把打包文件进行压缩
- `-f`：新的压缩包名称
- `-x`：解压文件
- `-v`：显示解压过程
**2.基础用法**
- 压缩文件/文件夹：`tar czf test.tgz test.txt`
- 解压文件/文件夹：`tar xzf test.tgz`
- 解压到指定目录：`tar xzf test.tgz -C 目录路径`



# Linux 常用命令速查 & 入门总结（适合新手）

这是一份针对 Linux 初学者的实用命令笔记，重点覆盖文件/目录操作、查看、复制移动删除、查看内容、压缩解压等日常最高频场景。

## 1. 导航与位置

| 命令    | 说明                                | 常用写法示例                                                                      |
| ----- | --------------------------------- | --------------------------------------------------------------------------- |
| `pwd` | 显示当前工作目录（Print Working Directory） | `pwd`                                                                       |
| `cd`  | 切换目录                              | `cd /etc` <br>`cd ..` <br>`cd ~` <br>`cd -`（返回上一个目录）                        |
| `ls`  | 列出目录内容                            | `ls` <br>`ls -a`（含隐藏文件）<br>`ls -lh`（人类可读大小 + 详情）<br>`ls -laR`（递归 + 详情 + 隐藏） |

**小技巧**：`ls -lah --color=auto` 几乎是所有人日常最爱用的组合。

## 2. 创建 / 删除

| 命令         | 说明                              | 常用示例                              |
|--------------|-----------------------------------|---------------------------------------|
| `touch`      | 创建空文件 / 更新文件时间戳       | `touch newfile.txt`                  |
| `mkdir`      | 创建目录                          | `mkdir mydir` <br>`mkdir -p deep/nested/dir`（递归创建） |
| `rmdir`      | 删除**空**目录                    | `rmdir empty_dir`                    |
| `rm`         | 删除文件或目录（**非常危险**！）   | `rm file.txt` <br>`rm -i file.txt`（逐个确认）<br>`rm -rf dir/`（强制递归删除，慎用！） |

**重要警告**：`rm -rf /` 或 `rm -rf /*` 会毁掉整个系统！新手建议先 alias rm='rm -i' 强制确认。

## 3. 复制、移动、重命名

| 命令   | 说明                  | 示例                                      |
|--------|-----------------------|-------------------------------------------|
| `cp`   | 复制文件/目录         | `cp file.txt backup/` <br>`cp -r dir1 dir2`（复制目录） |
| `mv`   | 移动 或 重命名        | `mv old.txt new.txt`（重命名）<br>`mv file.txt /tmp/`（移动） |

**常用选项**：`-i`（覆盖前询问）、`-f`（强制覆盖）、`-v`（显示过程）

## 4. 查看文件内容

| 命令     | 说明                              | 常用用法                                  |
|----------|-----------------------------------|-------------------------------------------|
| `cat`    | 显示全部内容                      | `cat file.txt` <br>`cat -n file.txt`（带行号） |
| `more`   | 分页查看（只能向下）              | `more long.log`                           |
| `less`   | 强大分页查看（上下翻页、搜索）    | `less /var/log/syslog` <br> 搜索：`/keyword` 然后 n/N |
| `head`   | 查看文件开头（默认10行）          | `head -n 20 access.log`                   |
| `tail`   | 查看文件结尾（最常用于看日志）    | `tail -n 50 error.log` <br>`tail -f access.log`（实时跟踪） |

**less 常用快捷键**：
- `/关键词` → 向下搜
- `?关键词` → 向上搜
- `n` / `N` → 下一个 / 上一个
- `q` → 退出

## 5. 查找与搜索

| 命令     | 说明                              | 示例                                      |
|----------|-----------------------------------|-------------------------------------------|
| `find`   | 在目录树中查找文件                | `find /home -name "*.txt"` <br>`find . -type d`（只找目录） |
| `grep`   | 在文本中搜索字符串                | `grep "error" app.log` <br>`grep -r "todo" .`（递归当前目录） |
| `which`  | 查找命令的可执行文件路径          | `which python`                            |
| `whereis`| 查找命令的二进制、源代码、手册    | `whereis ls`                              |

## 6. 权限与信息

- `chmod` 修改权限（r=4, w=2, x=1）
  - `chmod 755 script.sh`（拥有者全权，其他人可读可执行）
  - `chmod +x run.sh`（快速加执行权限）
- `chown` 改拥有者：`chown user:group file`
- `date` 显示/设置时间
  - `date` 
  - `date +"%Y-%m-%d %H:%M:%S"`
  - `date +%s`（时间戳）

## 7. 压缩与解压（最常用两种格式）

### zip / unzip（需安装）

```bash
# Ubuntu/Debian
sudo apt install zip unzip -y

# 压缩文件夹（必须加 -r）
zip -r myproject.zip myproject/

# 解压到指定目录
unzip myproject.zip -d destination/