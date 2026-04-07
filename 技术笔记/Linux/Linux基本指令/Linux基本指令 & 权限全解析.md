# Linux 常用命令入门 & 权限全解析

>**作者**：yuuki233233
>**目标**：德国 CS 本科 + 特斯拉软件工程师
>**适用人群**：大一/自学者，想快速上手 Linux 命令行 + 搞懂权限本质
>
>这是一份针对 Linux 初学者的实用命令笔记，重点覆盖文件/目录操作、查看、复制移动删除、查看内容、压缩解压等日常最高频场景。
>
>欢迎点赞、收藏、评论，一起卷 Linux！

## 1. 定位与路径（先学会找路）

### pwd - 当前工作目录

```bash
pwd # 输出示例：/home/yuuki/projects
```

### cd - 切换目录

```Bash
cd /etc          # 绝对路径
cd ..            # 上级
cd ~             # 家目录
cd -             # 上一个目录
```

### ls - 列出目录内容（最常用命令！）

```Bash
ls               # 列出当前目录可见文件
ls -a            # 显示隐藏文件（以 . 开头）
ls -l            # 详细列表（权限、所有者、大小、修改时间）
ls -lh           # 人类可读大小（K/M/G）
ls -la           # 详细 + 隐藏文件
ls -R            # 递归列出子目录
ls *.cpp         # 只看 .cpp 文件
```

**小技巧**：想看目录下最大文件？
```Bash
ls -lhS | head -n 5   # 按大小降序，前5个
```
**小技巧**：`ls -lah --color=auto` 几乎是所有人日常最爱用的组合。

## 2. 创建/删除/移动/复制

### touch - 创建空文件 / 更新时间戳

```Bash
touch newfile.txt
touch file1.txt file2.cpp
```

### mkdir - 创建目录

```Bash
mkdir newdir
mkdir -p a/b/c/d     # 递归创建多级目录
```

### rm - 删除文件/目录（小心！）

```Bash
rm file.txt          # 删除文件
rm -r dir            # 递归删除目录
rm -rf dir           # 强制递归删除（危险！慎用）
rm -i *.log          # 删除前询问确认
```

**安全写法**：先用 ls 确认要删的东西，再 rm。
**重要警告**：`rm -rf /` 或 `rm -rf /*` 会毁掉整个系统！新手建议先 `alias rm='rm -i'` 强制确认。

### cp - 复制文件/目录

```Bash
cp file.txt /backup/
cp -r dir /backup/   # 递归复制目录
cp -i file.txt dest/ # 覆盖前询问
```

### mv - 移动/重命名

```Bash
mv oldname.txt newname.txt   # 重命名
mv file.txt /new/path/       # 移动
mv -i file.txt dest/         # 覆盖前询问
```

**常用选项**：`-i`（覆盖前询问）、`-f`（强制覆盖）、`-v`（显示过程）

## 3. 查看文件内容

### cat - 查看文件内容

```Bash
cat file.txt
cat -n file.txt      # 显示行号
```

### less / more - 分页查看（大文件推荐 less）

```Bash
less log.txt         # 上下翻页，q 退出
```

### head / tail - 查看头/尾

```Bash
head -n 10 log.txt   # 前10行
tail -n 20 log.txt   # 后20行
tail -f log.txt      # 实时跟踪日志（开发最常用！）
```

**less 常用快捷键**：
- `/关键词` → 向下搜
- `?关键词` → 向上搜
- `n` / `N` → 下一个 / 上一个
- `q` → 退出

## 4. 查找与搜索

### find - 强大的文件查找

```Bash
find /home -name "*.cpp"          # 找所有 .cpp 文件
find . -type f -size +10M         # 找当前目录大于10MB的文件
find . -mtime -7                  # 7天内修改的文件
```

### grep - 文本搜索（神器！）

```Bash
grep "error" log.txt              # 查找包含 error 的行
grep -r "TODO" .                  # 递归搜索当前目录
grep -i "bug" *.cpp               # 忽略大小写
grep -n "warning" *.log           # 显示行号
```

## 5. 权限核心解析（重点）
### rwx 权限本质

| 对象  | r（读）     | w（写）          | x（执行/进入）     | 决定能否删除？ |
| --- | -------- | ------------- | ------------ | ------- |
| 文件  | 查看内容     | 修改内容          | 执行脚本/二进制     | 否       |
| 目录  | ls 查看文件名 | 创建/删除/重命名里面文件 | cd 进入、访问里面文件 | 是（w 决定） |

**关键结论**：文件能不能被删，只看它**所在目录**有没有 w 权限

### chmod 修改权限

符号法：
```Bash
chmod u+x script.sh
chmod go-rwx private/
```

数字法：
```Bash
chmod 755 script.sh    # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
```

### umask 计算

默认 umask 002：
- 文件：666 & 775 = 664（rw-rw-r--）
- 目录：777 & 775 = 775（rwxrwxr-x）

**为什么 umask 002？** 大部分文件不需要执行权限（源代码、日志、配置文件），所以默认去掉 o 的 x

### sudo / su / root

- `su -`：切换到 root，需要 root 密码
- `sudo`：临时提权，需要当前用户密码（推荐）
- `root`：无所不能

**小实验**：

```Bash
sudo touch /root/secret.txt
sudo chown $USER:$USER /root/secret.txt   # 改成你的
```

## 6. 压缩与解压（最常用两种格式）

### zip / unzip（需安装）

```bash
# Ubuntu/Debian
sudo apt install zip unzip -y

# 压缩文件夹（必须加 -r）
zip -r myproject.zip myproject/

# 解压到指定目录
unzip myproject.zip -d destination/
```

### tar（Linux 自带，最推荐）

```Bash
# 打包 + gzip 压缩（最常用 .tar.gz）
tar czvf backup.tar.gz folder/

# 解压
tar xzvf backup.tar.gz

# 解压到指定目录
tar xzvf backup.tar.gz -C /path/to/dir

# 只查看内容（不解压）
tar tvf backup.tar.gz
```

## 7. 其他高频小工具

- `man 命令` → 查看手册（按 q 退出）
- `alias ll='ls -lah' `→ 设置快捷别名（写到 ~/.bashrc 永久生效）
- `cal `→ 显示日历
- `bc` → 简单计算器
- `uname -a` → 查看系统信息
- `tree` → 显示目录树状结构（需安装：`sudo apt install tree`）
    - 推荐：`tree -C -L 3`（彩色 + 3 层）
- `df/du 磁盘空间`
	- ```Bash
		df -h           # 人类可读磁盘使用情况
		du -sh /dir     # 查看目录总大小
	  ```
- `ps/top/htop 进程查看`
	- ```Bash
		ps aux          # 查看所有进程
		top             # 实时监控（q 退出）
		htop            # 更友好版本（需安装）
	  ```
- `kill 杀死进程`
	- ```Bash
		kill 12345      # 温和终止
		kill -9 12345   # 强制杀死
	  ```
- `scp 远程复制文件`
	- ```Bash
		scp file.txt user@remote:/path/      # 本地 → 远程
		scp user@remote:/path/file.txt .     # 远程 → 本地
	  ```
## 安全小贴士（新手必看）

1. 永远不要随便用 `rm -rf /` 或 `rm -rf *`（尤其是 sudo）
2. 删除前先用 `ls` 确认目标
3. 重要操作加 `-i` 确认
4. 学习用 `man 命令` 或 `命令 --help` 获取最新准确帮助

代码仓库：[https://github.com/yuuki233233/cpp-learning-journey](https://github.com/yuuki233233/cpp-learning-journey) 
CSDN 主页：[https://blog.csdn.net/yuuki233233](https://blog.csdn.net/yuuki233233)

欢迎评论交流，一起卷 Linux ！