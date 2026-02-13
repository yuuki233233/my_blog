>**前言：**
>上篇中模拟实现了AVLTree和RBTree，这次就拿RBTree来封装实现set和map

# 源码及框架

- 下图框架，set（key） 和 map（key/value）没有直接写死，而是通过第二个模板参数 Value 决定_rb_tree_node中存储的数据类。 用 set 实例化 rb_tree，第二个模板参数实例化为 key，用 map 实例化 rb_tree，第二个模板参数实例化为 `pair<const key, T>`
- **注意**：源码里 `pair<const key, T>` ，T 代表 value，而内部写的 value_type 反而是红黑树节点中存储的数据类型
- **问题**：既然已经确定了_rb_tree_node中的类型，那么为什么还要写模板参数 key ？
	- 对 **insert** ：set 插入的是 key 对象，map 插入的是 pair 对象，可以用 Value 模板参数
	- 对 **find** ：set 查找的是 key 对象，map 查找的是 key。由于在 map 中， Value 是 pair 类型，所以不能用 Value 模板参数，需额外添加个 key 的模板参数
![](图片/QQ20260213-164357.png)
# 二、模拟实现 map 和 set
## 2.1 实现出复用红黑树的框架

























