>**前言：**
>在上两章中，学习了如何运用`set、map`和模拟实现二叉搜索树，从二叉搜索树过渡到AVL树，本质上就是在`k_value`的节点中多加了`平衡因子和旋转的概念`，旋转其实是为了让二叉树更加接近完美二叉树。
# 一、AVL树的概念
AVL树发明与平衡二叉树，在此之上引入了平衡因子的概念`(平衡因子：右子树的高度 - 左子树的高度)`，任何节点的平衡因子等于`0/1/-1`，AVL树整体节点数量和分布类似完全二叉树，高度控制在`O(logN)`，增删查改的效率也可控制在`O(logN)`，相比二叉搜索树有了本质提升
>平衡因子为0的AVL树是最完美的，但有些情况做不到高度差是0.比如一棵树是2节点，4节点等情况下，高度差最好是1，无法做到高度差是0
>![](图片/AVL树概念01.png)
>![](图片/AVL树概念02.png)
# 二、AVL树的实现
## 2.1 AVL树的结构
可以参照模拟实现二叉搜索树的代码进行对比
**二叉搜索树：**
```cpp
// 二叉搜索树节点结构
template<class K>
struct BSTNode
{
	K _key;               // 节点存储的关键码
	BSTNode<K>* _left;    // 左孩子节点指针
	BSTNode<K>* _right;   // 右孩子节点指针

	// 构造函数：初始化节点
	BSTNode(const K& key)
		:_key(key)
		, _left(nullptr)
		, _right(nullptr)
	{
	}
};
```

**AVL树：**
```cpp
template<class K, class V>
struct AVLTreeNode
{
	pair<K, V> _kv;
	AVLTreeNode<K, V>* _left;
	AVLTreeNode<K, V>* _right;
	AVLTreeNode<K, V>* _parent;
	int _bf;
	AVLTreeNode(const pair<K, V>& kv)
		:_kv(kv)
		,_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_bf(0)
	{ }
};
```
## 2.2 AVL树的插入
### 2.2.1 AVL树插入一个值的大概过程
1. 按二叉搜索树规则插入一个值
2. 更新平衡因子，更新完插入结束
3. 
### 2.2.2 更新平衡因子
## 2.3 旋转
### 2.3.1 旋转的原则
### 2.3.2 右单旋
### 2.3.3 右单旋代码实现
### 2.3.4 左单旋
### 2.3.5 左单旋代码实现

