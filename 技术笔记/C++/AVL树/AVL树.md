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
2. 更新平衡因子没有出现问题，更新完插入结束
3. 更新平衡因子过程中出现不平衡，需要利用旋转降低子树的高度
### 2.2.2 更新平衡因子
**更新原则：**
- 平衡因子 = 右子树高度 - 左子树高度
- 插入节点，会增加子树高度而影响当前平衡因子，节点插入右子树，parent的平衡因子++，反之亦然
- parent所在子树的高度是否变化决定了是否会继续往上更新
**更新停止条件：**
1. 更新后parent的平衡因子为0，节点的左子树和右子树一样高
2. 更新后parent的平衡因子由`0->1 / 0->-1`，插入前一样高的树，当平衡因子为1时，节点插入右子树，反之亦然
3. 更新后parent平衡因子为`2或-2`，平衡已被破坏，需要旋转降低树的高度
**更新到10节点，平衡因子为2，10所在的子树已不平衡，需要旋转处理**
![](图片/平衡因子01.png)

**更新到中间节点，3为根的子树高度不变，不会影响上一层，更新结束**
![](图片/平衡因子02.png)

**最![](图片/平衡因子03.png)坏更新到根停止**
### 2.2.3 插入节点及更新平衡因子的代码实现
```cpp
template<class K, class V>
bool insert(const K& key, const V& value)
{
	// 情况1：树为空
	if (_root == nullptr)
	{
		_root = new Node<K, V>;
		return tree;
	}

	// 情况2：树不为空
	Node* cur = _root;
	Node* parent = nullptr;
	while (cur) // 找到空节点
	{
		if (cur->_kv < key)
		{
			parent = cur;
			cur = cur->_right;
		}
		else if (cur->_kv > key)
		{
			parent = cur;
			cur = cur->_left;
		}
		else
		{
			return false;
		}
	}

	// 插入空节点
	cur = new Node(key, value);
	if (parent->_kv < key)
	{
		parent->_right = cur;
	}
	else
	{
		parent->_left = cur;
	}
	cur->_parent = parent;

	// 更新平衡因子
	while (parent)
	{
		// 更新平衡因子
		if (cur == parent->_left)
			parent->_bf--;
		else
			parent->_bf++;

		if (parent->_bf == 0)
		{
			// 平衡退出
			break;
		}
		else if (parent->_bf == 1 || parent->_bf == -1)
		{
			// 往上更新
			cur = parent;
			parent = parent->_parent;
		}
		else if(parent->_bf == 2 || parent->_bf == -2)
		{
			// 不平衡，旋转处理

		}
		else
		{
			assert(false);
		}
	}
	return true;
}
```
## 2.3 旋转
### 2.3.1 旋转的原则
- 保持搜索树的规则
- 平衡被破坏的树，用旋转降低高度
旋转总共分为四种：左单旋/右单旋/左右双旋/右左双旋
### 2.3.2 右单旋
**以下5张图解释了右单旋的抽象情况，可以解决大部分右单旋的问题**
![](图片/右单旋图1.png)
![](图片/右单旋图2.png)
![](图片/右单旋图3.png)
![](图片/右单旋图4.png)
![](图片/右单旋图5.png)
### 2.3.3 右单旋代码实现
### 2.3.4 左单旋
### 2.3.5 左单旋代码实现

