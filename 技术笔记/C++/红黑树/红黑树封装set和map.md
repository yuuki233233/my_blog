>**前言：**
>上篇中模拟实现了AVLTree和RBTree，这次就拿RBTree来封装实现set和map

# 源码及框架
```cpp

```
- 下图框架，set（key） 和 map（key/value）没有直接写死，而是通过第二个模板参数 Value 决定_rb_tree_node中存储的数据类。 用 set 实例化 rb_tree，第二个模板参数实例化为 key，用 map 实例化 rb_tree，第二个模板参数实例化为 `pair<const key, T>`
- **注意**：源码里 `pair<const key, T>` ，T 代表 value，而内部写的 value_type 反而是红黑树节点中存储的数据类型
- 
![](图片/QQ20260213-164357.png)