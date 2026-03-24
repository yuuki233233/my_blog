## 下载 .git

- CentOS：`sudo apt install git`
- ubanto：`sudo yum install git`
- 查看 `.git` 版本：`git --version`

## 配置 .git

| 命令                                  | 作用              |
| ----------------------------------- | --------------- |
| `git init`                          | 创建 `.git`       |
| `git config user.name "you name"`   | 给 `.git` 配置仓库名  |
| `git config user.emain "you email"` | 给 `.git` 配置仓库地址 |
| `git config --unset user.name`      | 删除 `.git` 仓库名   |
| `git config --unset user.email`     | 删除 `.git` 仓库地址  |
| `git config -l`                     | 查看 `.git` 配置信息  |

## 修改文件

| 命令                      | 作用             |
| ----------------------- | -------------- |
| `git clone "HTTPS/SSH"` | 拷贝仓库           |
| `git add [文件名1] [文件名2]` | 添加单个文件或多个文件    |
| `git add .`             | 添加所有文件到暂存区     |
| `git commit -m "修改信息"`  | 将文件从暂存区写入到本地仓库 |
| `git status`            | 查看当前仓库状态       |

Git 追踪管理的其实是修改，而不是文件
>修改：新增、修改、删除

当我们修改 `ReadMe` 文件时，查看 `ReadMe` 的状态，用 `git status`
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   ReadMe  # 修改文件，在工作区中
#
no changes added to commit (use "git add" and/or "git commit -a")
```

显示暂存区和工作区文件的差异用： `git diff [file]`
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git diff ReadMe
diff --git a/ReadMe b/ReadMe
index 15fb616..f7a9997 100644
--- a/ReadMe
+++ b/ReadMe
@@ -1,2 +1,3 @@
 hello Git
 hello linux
+hello world # 改动后的第二行开始
```

差异文件添加后再用 `git status` 查看状态
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
# Changes to be committed: # 需要用 commit 添加到本地仓库
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   ReadMe
#
```

`commit` 后的状态
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git commit -m "modify ReadMe"
[master cd886dc] modify ReadMe
 1 file changed, 1 insertion(+)
 
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
nothing to commit, working directory clean # 暂存区干净了
```

## 查看 .git 文件

命令：`git log`
```bash
17 directories, 24 files
[zjh@izj6cd3zr5adf6jnm3mwihz .git]$ git log
commit e6a643c5c2d60d07214cb01e12c5612546524667
Author: yuuki233233 <yuuki233233@outlook.com>
Date:   Mon Mar 23 16:32:41 2026 +0800

    add 3 file # 我们提交的三个文件

commit 0b111c915a7a8e621565eeb9cf6a9d96db718bf3 # 文件哈希值（可在objects查找文件）
Author: yuuki233233 <yuuki233233@outlook.com> # 仓库名称和邮箱
Date:   Mon Mar 23 16:30:22 2026 +0800        # 提交时间

	add first fiile # 我们提交的第一份文件
```

上面提交信息不好看，可以使用：`git log --pretty=oneline`
```bash
# 只有哈希值和提交文件
e6a643c5c2d60d07214cb01e12c5612546524667 add 3 file       # 提交的三份文件
0b111c915a7a8e621565eeb9cf6a9d96db718bf3 add first fiile  # 提交的第一份文件
```

修改的工作区内容会写入对象库的一个性的 git 对象中
```bash
.
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── HEAD # 指针，指向 heads/master
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── prepare-commit-msg.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   └── update.sample
├── index # 暂存区
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects  # 文件维护(哈希值)
│   ├── 0b
│   │   └── 111c915a7a8e621565eeb9cf6a9d96db718bf3
│   ├── 3f
│   │   └── daafec4cd1f8c429c0de092492c017c6064484
│   ├── 7b
│   │   └── 5bbd989152e5bab6b5476f50133e16137d6b30
│   ├── af
│   │   └── 6a2b2b4891c7e177f543386efd146748aaec38
│   ├── e6
│   │   ├── 9de29bb2d1d6434b8b29ae775ad8c2e48c5391
│   │   └── a643c5c2d60d07214cb01e12c5612546524667
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```

## 版本回退

在写 `ReadMe` 文件时，有三个版本
```
version 1
hello Git

version 2
hello Git
hello linux

version 3
hello Git
hello linux
hello world
```

我们想回退到 version 1 时，需要用到：`git reset [--soft | mixed | --hard] [HEAD]]`
- --soft：回退版本库中的内容，对工作区和暂存区无影响
- --mixed：回退版本库和暂存区的内容，对工作区没影响（默认选项）
- --hard：回退版本库和暂存区和工作区（谨慎使用）

| 选项      | 工作区 | 暂存区 | 版本库 |
| ------- | --- | --- | --- |
| --soft  | No  | No  | Yes |
| --mixed | No  | Yes | Yes |
| --hard  | Yes | Yes | Yes |

**第一种情况，没用退出Xshell，在屏幕里能找到id**
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git log --pretty=oneline
cd886dc6d233d048393492ca59f4d03f84acc863 modify ReadMe   # version 3
8596745dc113f87d93bf7b61251f04c0ff71a831 add hello linux # version 2
e6a643c5c2d60d07214cb01e12c5612546524667 add 3 file
0b111c915a7a8e621565eeb9cf6a9d96db718bf3 add first fiile # version 1

# 我们想回退到 version 3
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset --hard 0b111c915a7a8e621565eeb9cf6a9d96db718bf3
HEAD is now at 0b111c9 add first fiile

# 查看当前历史，前面的 version 2/3 和 3 file 都没了
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git log --pretty=oneline
0b111c915a7a8e621565eeb9cf6a9d96db718bf3 add first fiile

# 还可以回退到 version 3
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset --hard cd886dc6d233d048393492ca59f4d03f84acc863
HEAD is now at cd886dc modify ReadMe

# 又回来了
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git log --pretty=oneline
cd886dc6d233d048393492ca59f4d03f84acc863 modify ReadMe
8596745dc113f87d93bf7b61251f04c0ff71a831 add hello linux
e6a643c5c2d60d07214cb01e12c5612546524667 add 3 file
0b111c915a7a8e621565eeb9cf6a9d96db718bf3 add first fiile

```

**第二种情况，通过 git reflog 找到 id**
```bash
# 回退版本
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset --hard 0b111c915a7a8e621565eeb9cf6a9d96db718bf3
HEAD is now at 0b111c9 add first fiile

# git 历史（没用回退前的version3 id）
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git log
commit 0b111c915a7a8e621565eeb9cf6a9d96db718bf3
Author: yuuki233233 <yuuki233233@outlook.com>
Date:   Mon Mar 23 16:30:22 2026 +0800

    add first fiile

# 通过 git reflog 来查找 version3 id
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reflog
0b111c9 HEAD@{0}: reset: moving to 0b111c915a7a8e621565eeb9cf6a9d96db718bf3
cd886dc HEAD@{1}: reset: moving to cd886dc6d233d048393492ca59f4d03f84acc863
0b111c9 HEAD@{2}: reset: moving to 0b111c915a7a8e621565eeb9cf6a9d96db718bf3
cd886dc HEAD@{3}: commit: modify ReadMe
8596745 HEAD@{4}: commit: add hello linux
e6a643c HEAD@{5}: commit: add 3 file
0b111c9 HEAD@{6}: commit (initial): add first fiile

# 继续回退
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset --hard cd886dc
HEAD is now at cd886dc modify ReadMe

# 回退成功
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git log --pretty=oneline
cd886dc6d233d048393492ca59f4d03f84acc863 modify ReadMe
8596745dc113f87d93bf7b61251f04c0ff71a831 add hello linux
e6a643c5c2d60d07214cb01e12c5612546524667 add 3 file
0b111c915a7a8e621565eeb9cf6a9d96db718bf3 add first fiile
```

## 撤销修改

| 情况               | 工作区      | 暂存区      | 版本库      | 解决方式 |
| ---------------- | -------- | -------- | -------- | ---- |
| 情况1：只有工作区有内容     | xxx code |          |          |      |
| 情况2：只有工作区和暂存区有内容 | xxx code | xxx code |          |      |
| 情况3：都有内容         | xxx code | xxx code | xxx code |      |

### 情况1：只有工作区修改内容
**没添加到暂存区的文件回退**
1. 手动撤销，不推荐，容易出错
2. 使用 `git checkout -- [文件]` 推荐，如下：
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git diff ReadMe
diff --git a/ReadMe b/ReadMe
index f7a9997..a463423 100644
--- a/ReadMe
+++ b/ReadMe
@@ -1,3 +1,4 @@
 hello Git
 hello linux
 hello world
+hello zjh   # 修改后文件的内容

[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git checkout -- ReadMe # 回退命令

[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ cat ReadMe # 回退完成
hello Git
hello linux
hello world
```

### 情况2：已经添加到暂存区
将工作区和暂存区返回到版本库的内容：
- 回退到上个版本：使用 `git reset HEAD （默认使用--mixed，回退暂存区和版本库，也可以用 --haed）`
```bash
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ cat ReadMe
hello Git
hello linux
hello world
hello zjh   # 添加新内容

# 从工作区添加到暂存区
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git add ReadMe

# 查看 git 状态
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   ReadMe
#

# 使用 git reset HEAD [file] 把 Add 状态回退到没有 Add 状态
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset HEAD ReadMe
Unstaged changes after reset:
M	ReadMe

# 查看状态
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   ReadMe
#
no changes added to commit (use "git add" and/or "git commit -a")

# 回退到情况1，用 git checkout -- [file] 进行回退
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git checkout -- ReadMe

# 查看状态
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
nothing to commit, working directory clean

# hello zjh 没有了，回退成功
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ cat ReadMe
hello Git
hello linux
hello world
```

### 情况3：版本库中把最新版回退到上一版

**前提条件，远程仓库没用版本库的最新代码**
```bash
# 写文件
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ vim ReadMe

## 修改文件
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ cat ReadMe
hello Git
hello linux
hello world
hello zjh

# 添加到暂存区
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git add ReadMe

# 添加到版本库
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git commit -m "modify ReadMe"
[master 1affcca] modify ReadMe
 1 file changed, 1 insertion(+)

# 用git reset -- hard HEAD^ [file] 回退到上个版本，HEAD^表示上一个版本
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git reset --hard HEAD^
HEAD is now at cd886dc modify ReadMe

# 查看状态，结果很干净
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ git status
# On branch master
nothing to commit, working directory clean

# 回退成功，没有 hello zjh
[zjh@izj6cd3zr5adf6jnm3mwihz gitcode]$ cat ReadMe
hello Git
hello linux
hello world
```

## 删除文件
