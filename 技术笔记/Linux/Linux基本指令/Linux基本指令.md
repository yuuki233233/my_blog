
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


>**作者**：yuuki233233
>**目标**：德国 CS 本科 + 特斯拉软件工程师
>**适用人群**：零基础想快速上手 Linux 命令行的大一/自学者
>**学习建议**：先在 Ubuntu 虚拟机或云服务器上手敲 3-5 遍，边敲边记笔记


# Linux 常用命令速查 & 入门总结（适合新手）

这是一份针对 Linux 初学者的实用命令笔记，重点覆盖文件/目录操作、查看、复制移动删除、查看内容、压缩解压等日常最高频场景。

## 1. 定位与路径相关（先学会找路）

### pwd - 显示当前工作目录
```bash
pwd # 输出示例：/home/yuuki/projects
```

### cd - 切换目录

```Bash
cd /etc          # 绝对路径
cd ..            # 上级目录
cd ~             # 回到家目录（等价于 cd）
cd -             # 回到上一个目录
cd               # 直接回 ~（家目录）
```

### ls - 列出目录内容（最常用命令！）

```Bash
ls               # 列出当前目录可见文件
ls -a            # 显示隐藏文件（以 . 开头的）
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

## 2. 创建 / 删除

| 命令      | 说明                 | 常用示例                                                                  |
| ------- | ------------------ | --------------------------------------------------------------------- |
| `touch` | 创建空文件 / 更新文件时间戳    | `touch newfile.txt`                                                   |
| `mkdir` | 创建目录               | `mkdir mydir` <br>`mkdir -p deep/nested/dir`（递归创建）                    |
| `rmdir` | 删除**空**目录          | `rmdir empty_dir`                                                     |
| `rm`    | 删除文件或目录（**非常危险**！） | `rm file.txt` <br>`rm -i file.txt`（逐个确认）<br>`rm -rf dir/`（强制递归删除，慎用！） |

**重要警告**：`rm -rf /` 或 `rm -rf /*` 会毁掉整个系统！新手建议先 `alias rm='rm -i'` 强制确认。

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

## 8. 其他高频小工具

- `man 命令` → 查看手册（按 q 退出）
- `alias ll='ls -lah' `→ 设置快捷别名（写到 ~/.bashrc 永久生效）
- `cal `→ 显示日历
- `bc` → 简单计算器
- `uname -a` → 查看系统信息
- `tree` → 显示目录树状结构（需安装：`sudo apt install tree`）
    - 推荐：`tree -C -L 3`（彩色 + 3 层）

## 安全小贴士（新手必看）

1. 永远不要随便用 `rm -rf /` 或 `rm -rf *`（尤其是 sudo）
2. 删除前先用 `ls` 确认目标
3. 重要操作加 `-i` 确认
4. 学习用 `man 命令` 或 `命令 --help` 获取最新准确帮助
5. 多练习！建议在虚拟机或 WSL 上随便折腾