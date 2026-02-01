>**前言：**
>在掌握二叉搜索树（BST）的基础上，我们发现其存在显著缺陷：若插入数据有序，二叉搜索树会退化为单链表，此时增删查改的时间复杂度从 O(logN) 骤降为 O(N)。为解决这一问题，G.M.Adelson-Velsky 和 E.M.Landis 在 1962 年提出了**AVL 树**—— 一种自平衡的二叉搜索树。
>
AVL 树的核心是在二叉搜索树的基础上引入「平衡因子」和「旋转」机制，强制让树的任意节点左右子树高度差不超过 1，从而将树的高度严格控制在 O(logN)，保证所有操作的时间复杂度稳定在 O(logN)。
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
模拟实现左单旋和右单旋最重要的就是画图
```cpp
// 右旋
void RotateR(Node* parent)
{
	// 最核心部分
	/*Node* SubL = parent->_left;
	Node* SubLR = SubL->_right;

	parent->_left = SubLR;
	SubL->_right = parent;*/

	
	// 核心三节点，不需要其他节点
	Node* SubL = parent->_left;
	Node* SubLR = SubL->_right;


	parent->_left = SubLR;		// 1.需画图理解

	// 对应h = 0
	if(SubLR != nullptr)
		SubLR->_parent = parent;	// 2.要满足父节点(左指针或右指针) -> 子节点，子节点(父指针) -> 父节点

	SubL->_right = parent;		// 满足 1.
	parent->_parent = SubL;		// 满足 2.


	Node* pParent = parent->_parent;
	if (parent == _root)	/*根为根节点*/
	{
		_root = SubL;
		SubL->_parent = nullptr;
	}
	else	/*根为子树*/
	{
		if (pParent->_left == parent)
		{
			pParent->_left = SubL;
		}
		else
		{
			pParent->_right = SubL;
		}
	}
	
	// 旋转完，更新平衡因子
	SubL->_bf = parent->_bf = 0;
}
```
### 2.3.4 左单旋代码实现
```cpp
// 左旋
void RotateL(Node* parent)
{
	// 核心三节点，不需要其他节点
	Node* SubR = parent->_right;
	Node* SubRL = SubR->_left;

	parent->_right = SubRL;
	SubR->_left = parent;
	parent->_parent = SubR;
	if(SubRL != nullptr)
		SubRL->_parent = parent;

	Node* pParent = parent->_parent;
	if (parent == _root)
	{
		_root = SubR;
		SubR->_parent = nullptr;
	}
	else
	{
		if (pParent->_left == parent)
		{
			pParent->_left = SubR;
		}
		else
		{
			pParent->_right = SubR;
		}
	}

	parent->_bf = SubR->_bf = 0;
}
```
### 2.3.5 左右双旋
图下右单旋和左右双旋的比较可进行总结：
**右单旋 / 左单旋：** 节点之间成一条直线（右单旋只解决纯粹的左边搞）
**左右双旋 / 右左双旋：** 节点之间成一条曲折的线（但在b子树不单纯是左边高，只是对于10是左边高，对于5是右边高，需旋转两次才能解决。对5为旋转点进行一次左单旋，对10为旋转点进行一次右单旋）
![](图片/左右双旋.png)
 >**对于右单旋：** 旋转结束，各节点平衡因子都为0
 >**对于左右双旋：** 旋转结束，原父节点**左右两子节点平衡因子不一定全为0**，左子节点平衡因子可能为：1、0、-1，右子节点平衡因子可能为：0、1、-1，这时由高度不同会出现3个场景，由下图可知：
 >![](图片/左右双旋结果图01.png)
 >![](图片/左右双旋结果图02.png)
 >**场景1**：`h>=1`时，新增节点插入在e子树，e子树高度从`h-1`并不断更新`8->5->10`平衡因子，引发旋转，其中8的平衡因子为-1，旋转后8和5平衡因子为0，10平衡因子为1
 >**场景2**：`h>=1`时，新增节点插入在f子树，f子树高度从`h-1`变为h并不断更新`8->5->10`平衡因子，引发旋转，其中8的平衡因子为-1，旋转后8和5平衡因子为0，10平衡因子为-1
 >**场景3**：`h==0`时，a/b/c都是空树，b自己就是一个新增节点，不断更新`5->10`平衡因子，引发旋转，其中8的平衡因子为0，旋转后8和10和5平衡因子均为0
### 2.3.6 模拟实现左右双旋
```cpp
// 左右双旋
void RotateLR(Node* parent)
{
	Node* subL = parent->_left;
	Node* subLR = subL->_right;
	int bf = subLR->_bf;

	RotateL(parent->_left);
	RotateR(parent);

	if (bf == 0)
	{
		subL->_bf = 0;
		subLR->_bf = 0;
		parent->_bf = 0;
	}
	else if (bf == -1)
	{
		subL->_bf = 0;
		subLR->_bf = 0;
		parent->_bf = 1;
	}
	else if (bf == 1)
	{
		subL->_bf = -1;
		subLR->_bf = 0;
		subLR->_bf = 0;
	}
	else
	{
		assert(false);
	}
}
```
### 2.3.7 模拟实现右左双旋
```cpp
// 右左双旋
void RotateRL(Node* parent)
{
	Node* subR = parent->_right;
	Node* subRL = subR->_left;
	int bf = subRL->_bf;

	if (bf == 0)
	{
		subR->_bf = 0;
		subRL->_bf = 0;
		parent->_bf = 0;
	}
	else if(bf == -1)
	{
		subR->_bf = 0;
		subRL->_bf = 0;
		parent->_bf = -1;
	}
	else if (bf == 1)
	{
		subR->_bf = 1;
		subRL->_bf = 0;
		subRL->_bf = 0;
	}
	else
	{
		assert(false);
	}
}
```
## 2.4 AVL树的查找
那二叉搜索树逻辑实现即可，搜索效率`O(lngN)`
```cpp
Node* Find(const k& key)
{
	Node* cur = _root;
	while(cur)
	{
		if(cur->_kv.frist < key)
		{
			cur = cur->_right;
		}
		else if(cur->_kv.frist > key)
		{
			cur = cur->_left;
		}
		else
		{
			return cur;
		}
	}
	
	return nullptr;
}
```
## 2.5 AVL树平衡检测
我们实现的AVL是否合格，我们通过检查左右子树高度差的程序进行反向验证，同时检查一下节点的平衡因子更新是否出现了问题
```cpp
int _Height(Node* root)
{
	if(root == nullptr)
		return 0;
		
	int leftHeight = _Height(root->_left);
	int rightHeight = _Height(root->_right);
	
	return leftHeight > rightHeight ? leftHeight + 1 : rightHeight + 1;
}

bool _IsBanlanceTree(Node* root)
{
	// 空树也是AVL树
	if(nullptr == root)
		return true;
		
	// 计算pRoot结点的平衡因⼦：即pRoot左右⼦树的⾼度差
	int leftHeight = _Height(root->_left);
	int rightHeight = _Height(root->_right);
	int diff = rightHeight - leftHeight;
	
	
	// 如果计算出的平衡因⼦与pRoot的平衡因⼦不相等，或者
	// pRoot平衡因⼦的绝对值超过1，则⼀定不是AVL树
	if(abs(diff) >= 2)
	{
		cout << root->_kv.first << "⾼度差异常" << endl;
		return false;
	}
	
	if(root->_bf != diff)
	{
		cout << root->kv.frist << "平衡因子异常" << endl;
		return false;
	}
	
	// pRoot的左和右如果都是AVL树，则该树⼀定是AVL树
	return _IsBalanceTree(root->_left) && _IsBalanceTree(root->_right);
}
```
# 三、总结