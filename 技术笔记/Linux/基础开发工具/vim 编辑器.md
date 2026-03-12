# vim 概念
vim 编辑器分为三种模式：命令模式（command mode）、插入模式（insert mode）、末行模式（last line mode）
```text
插入模式 <-> 命令模式 <-> 末行模式
```

- 命令模式（command mode）：
	- 按 `i（insert）`：进入 `插入模式（insert mode）`进行字符写入
	- 按 `按住 Shift 和 ；`：进入 `末行模式（last line mode）`进行保存或退出
	- 控制屏幕光标移动，删除字符、移动复制某区段及进入 insert mode 下
- 插入模式（insert mode）：
	- 可进行**文字输入**
	- 按 `Esc` 回到命令模式
- 末行模式（last line mode）
	- 文件保存或退出，也可以进行文件替换，找字符串、列出行号等
	- 按 `Esc` 回到命令模式

# vim 基本操作
## 移动光标（命令模式下）
左下上右：`h(左) j(下) k(上) l(右)`
顶部：`gg`
底部：`G`
指定行列：`[选定行数] + G`
跳跃单个单词：`w(左) b(右)`
跳跃多个单词：`[跳跃单词数] w`

