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

## 基本 .git 命令

| 命令                      | 作用           |
| ----------------------- | ------------ |
| `git clone "HTTPS/SSH"` | 拷贝仓库         |
| `git add [文件名1] [文件名2]` | 添加单个文件或多个文件  |
| `git add .`             | 添加所有文件到暂存区   |
| `git commit -m "修改信息"`  | 将文件从暂存区添加到仓库 |

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



```bash
.
├── branches
├── COMMIT_EDITMSG
├── config
├── description
├── HEAD
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
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects  # 文件存放(哈希值)
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