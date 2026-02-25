>**作者**：yuuki233233
>**时间：** 2026年2月
>**目标**：德国CS本科 + 特斯拉软件工程师
>**前言**：
>上篇手撕了RBTree单树，这次继续卷封装set和map。整个过程花了10+小时debug和整理，代码基本完整可跑（已测试insert、迭代器遍历、[]插入/修改）
> 代码仓库：[https://github.com/yuuki233233/learning-journey](https://github.com/yuuki233233/learning-journey) 
> CSDN 主页：[https://blog.csdn.net/yuuki233233](https://blog.csdn.net/yuuki233233)


# 一、源码及框架（为什么需要KeyOfT）
- 下图框架，`set（key）` 和 `map（key/value）`没有直接写死，而是通过第二个模板参数` Value` 决定`_rb_tree_node`中存储的数据类。 用 `set` 实例化 `rb_tree`，第二个模板参数实例化为 `key`，用 `map` 实例化 `rb_tree`，第二个模板参数实例化为 `pair<const key, T>`
- **注意**：源码里 `pair<const key, T>` ，`T` 代表 `value`，而内部写的 `value_type` 反而是红黑树节点中存储的数据类型
- **问题**：既然已经确定了 `_rb_tree_node` 中的类型，那么为什么还要写模板参数 `key` ？
	- 对 **insert** ：`set` 插入的是 `key` 对象，`map` 插入的是 `pair` 对象，可以用 `Value` 模板参数
	- 对 **find** ：`set` 查找的是 `key` 对象，`map` 查找的是 `key`。由于在 `map` 中， `Value` 是 `pair` 类型，所以不能用 `Value` 模板参数，需额外添加个 `key` 的模板参数
![](图片/QQ20260213-164357.png)
**总结**：
 >- **set**：存储 `key`，比较直接用` `key``。
 >- **map**：存储 `pair`，比较只看 `first(key)`，不能直接用 `pair` 比较。
 >- **解决方案**：引入 **KeyOfT仿函数**（提取 key 的接口），统一比较逻辑。
 >- **模板参数设计**：
 >	- **RBTree**：`K` 是 `key` 类型，`T` 是节点存储类型（`set` 为 `K`，`map` 为 `pair`），`KeyOfT` 决定怎么从 `T` 取 `key`。
# 二、核心代码实现
## 2.1 RBTree节点 & 基本结构
```cpp
enum Colour
{
	RED,
	BLACK
};

/*
* 这里的 RBTree 里的模板参数要跟 _stl_tree 保持一致
* 原本为固定的数据类型 pair<K, V> 改成 泛型 T _data;
 */
template<class T>
struct RBTreeNode
{
	T _data;              // 键值对
	RBTreeNode<T>* _left;     // 左孩子
	RBTreeNode<T>* _right;    // 右孩子
	RBTreeNode<T>* _parent;   // 父节点（红黑树必须维护父节点）
	Colour _col;                 // 节点颜色

	// 构造函数
	RBTreeNode(const T& data)
		:_data(data)
		, _left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _col(RED)// 默认红（插入时默认红，减少黑高破坏）
	{}
};

/*
* 在原模板的基础上，新添加个模板类似，该模板类型是仿函数
 */
template<class K, class T, class KeyOfT>
class RBTree
{
	typedef RBTreeNode<T> Node;
public:
private:
	Node* _root = nullptr;
};
```
## 2.2 RBTree模板类（Insert核心 + 旋转）
- 在 `stl_map` 中，第二个模板参数用的是 `T`，改为 `V（Value）`会更好
- `set` 直接用`data(key)` 比较是没有问题的，但是 `map` 不能用 `pair` 进行比较，`map` 的比较要用到 `data.first(pair<K, V>._kv.first)`，但 `set` 又不支持用 `first`。如何解决：需另外用到 `仿函数(KeyOfT)`
```cpp

// 左旋转：以parent为旋转点，右孩子上位
void RotateL(Node* parent)
{
    Node* subR = parent->_right;    // 失衡节点的右孩子（新根）
    Node* subRL = subR->_left;      // 新根的左子树（需要转移）
    Node* pParent = parent->_parent; // 失衡节点的父节点

    // 1. 转移subRL：挂到parent的右子树
    parent->_right = subRL;
    if (subRL)
        subRL->_parent = parent;

    // 2. 父节点降级：parent作为subR的左孩子
    subR->_left = parent;
    parent->_parent = subR;

    // 3. 链接新根到原父节点
    if (pParent == nullptr)
    {
        _root = subR;
        subR->_parent = nullptr;
    }
    else
    {
        if (pParent->_left == parent)
            pParent->_left = subR;
        else
            pParent->_right = subR;
        subR->_parent = pParent;
    }

    // 4. 重置平衡因子
    parent->_bf = subR->_bf = 0;
}

// 右旋转：以parent为旋转点，左孩子上位
void RotateR(Node* parent)
{
    Node* subL = parent->_left;    // 失衡节点的左孩子（新根）
    Node* subLR = subL->_right;    // 新根的右子树（需要转移）
    Node* pParent = parent->_parent; // 失衡节点的父节点

    // 1. 转移subLR：挂到parent的左子树
    parent->_left = subLR;
    if (subLR)
        subLR->_parent = parent;

    // 2. 父节点降级：parent作为subL的右孩子
    subL->_right = parent;
    parent->_parent = subL;

    // 3. 链接新根到原父节点
    if (pParent == nullptr)
    {
        // 原parent是根节点，更新根
        _root = subL;
        subL->_parent = nullptr;
    }
    else
    {
        if (pParent->_left == parent)
            pParent->_left = subL;
        else
            pParent->_right = subL;
        subL->_parent = pParent;
    }

    // 4. 重置平衡因子（旋转后子树高度恢复，BF归0）
    parent->_bf = subL->_bf = 0;
}

// 插入
bool Insert(const T& data)
{
	if (_root == nullptr)
	{
		_root = new Node(data);
		_root->_col = BLACK;

		return true;
	}

	/*
	* set 直接用 data(key) 比较是没有问题的，但是 map 不能用 pair 进行比较
	* map 的比较要用到 data.first(pair<K, V>._kv.first)，但 set 又不支持用 first
	*
	* 如何解决：需另外用到仿函数(KeyOfT)
	 */
	KeyOfT kot;
	Node* parent = nullptr;
	Node* cur = _root;
	while (cur)
	{

		if (kot(cur->_data) < kot(data))
		{
			parent = cur;
			cur = cur->_right;
		}
		else if (kot(cur->_data) > kot(data))
		{
			parent = cur;
			cur = cur->_left;
		}
		else
		{
			return false;
		}
	}

	cur = new Node(data);
	cur->_col = RED;	// 新节点为红色
	if (kot(parent->_data) < kot(data))
		parent->_right = cur;
	else
		parent->_left = cur;
	// 链接父亲
	cur->_parent = parent;

	// 父亲是红色，出现连续的红色节点（需处理）
	while (parent && parent->_col == RED) // 分两种情况：1.叔叔在左边 2.叔叔在右边
	{	// 条件parent：防止空指针（_root节点的父亲为NULL）
		Node* grandfater = parent->_parent;

		if (parent == grandfater->_left) // 叔叔在右边
		{
			//   g
			// p   u
			Node* uncle = grandfater->_right;
			if (uncle && uncle->_col == RED)	// 叔叔存在且为红色（变色）
			{
				parent->_col = uncle->_col = BLACK;
				grandfater->_col = RED;

				// 继续向上处理，最坏结果处理到根
				cur = grandfater;
				parent = cur->_parent;
			}
			else // 叔叔不存在，或存在且为黑（旋转+变色）
			{
				if (cur == parent->_left) // c在父亲左边，构成直线，只单旋一次
				{
					//     g
					//   p   u
					// c
					RotateR(grandfater);
					parent->_col = BLACK;
					grandfater->_col = RED;
				}
				else// c在父亲右边，构成折现，需要双旋
				{
					//     g
					//   p   u
					//    c
					RotateL(parent);
					RotateR(grandfater);
					cur->_col = BLACK;
					grandfater->_col = RED;
				}

				break;
			}
		}
		else // 叔叔在左边（类似上列代码）
		{
			//   g
			// u   p
			Node* uncle = grandfater->_left;

			// 叔叔存在且为红（变色即可）
			if (uncle && uncle->_col == RED)
			{
				parent->_col = uncle->_col = BLACK;
				grandfater->_col = RED;

				// 继续向上处理
				cur = grandfater;
				parent = cur->_parent;
			}
			else// 叔叔不存在，或存在且为黑（旋转+变色）
			{
				//     g
				//   u   p
				//         c
				if (cur == parent->_right)
				{
					RotateL(grandfater);
					parent->_col = BLACK;
					grandfater->_col = RED;
				}
				else
				{
					//     g
					//   u   p
					//      c
					RotateR(parent);
					RotateL(grandfater);
					cur->_col = BLACK;
					grandfater->_col = RED;
				}

				break;
			}
		}
	}
	_root->_col = BLACK; // _root节点必为BLACK

	return true;
}
```
## 2.3 迭代器实现（++ / -- / * / ->）
**iterator实现思路分析：**
- `list` 和 `RBTree` 本质上是各个节点链接而成，但不同的是链接的方式不同。`list`：{节点 -> 节点}，`RBTree`：{节点} -> {节点1，节点2}
- `iterator` 实现的框架于 `list` 的 `iterator` 思路类似，用一个类型封装节点的指针，在通过**重载运算符实现**，迭代器像指针一样访问
- `operator++` 和 `operator--` 是实现 `iterator` 迭代器的难点。首先是找 `begin() 最左节点 `与 `end()最右节点`，其次`再是控制局部遍历`
- **关于operator++：**
	- 当 `cur（所在节点）的左子树被访问完`，`cur` 本身也被访问了，于是访问 `cur 下一个节点（右节点）的最左节点`
	- 当` cur（所在节点）的右子树被访问完`，`cur` 本身也被访问了，这就代表 `cur 为根的一整棵树被访问完了`，于是要到 cur 的父亲节点
		- **cur 在父亲节点的左**：访问完了 `cur` 一整棵树，由于 `cur` 在父亲的左节点，代表父亲的左子树遍历完了，于是到父亲节点即可
		- **cur 在父亲节点的右**：访问完了 `cur` 一整棵树，由于 `cur` 在父亲的右节点，代表父亲的右子树遍历完了，那父亲节点也遍历完了，于是需要访问父亲节点的祖父节点
	- 当 `cur` 访问到根节点，如果 `cur` 所在根节点的父节点为空，那么就访问完了整棵树`it->nullptr == end`
- **关于end()**：我们看下图，如果 it 指向50时，++it 指向40，直到循环遍历到18，根节点18没有父亲，就把 it 中的节点指针置为 nullptr，用 nullptr 去充当 end。需要注意的是stl源码空，红黑树增加了一个哨兵位头结点做为 end()，这哨兵位头结点和根互为父亲，左指向最左结点，右指向最右结点。相比我们用 nullptr 作为 end()，差别不大，他能实现的，我们也能实现。只是 -end() 判断到结点时空，特殊处理一下，让迭代器结点指向最右结点。具体参考迭代器--实现。
- operator++ 访问顺序为 左子树 -> 根节点 -> 右子树，operator-- 访问顺序为 右子树 -> 根节点 -> 左子树
- set 的 iterator 不支持修改，把 set 的第二个模板参数改成 const K 即可，`RBTree<K,const K, SetKeyOfT> _t;`需要注意有坑，详细部分看代码解析
- map 的 iterator 不支持修改 key，但是可以修改 value，把第二个模板参数 pair 的第一个参数改成 const K 即可，`RBTree<K, pair<const K, V>, MapKeyOfT> _t;`
- 还要很多重要的细节，如`1.支持[]的插入、查找和修改  2.反向迭代器的注意事项  3.模板内嵌的处理`等等，请看下列从零开始模拟实现RBTree封装Myset和Mymap
- ![](图片/一颗红黑树.png)
```cpp
/*
* 考虑到有 iterator 和 const_iterator 两种迭代器
* 需要传 Ref 和 Ptr 模板参数
 */
template<class T, class Ref, class Ptr>
struct RBTreeIterator
{
	typedef RBTreeNode<T> Node;
	typedef RBTreeIterator<T, Ref, Ptr> Self;

	Node* _node;
	Node* _root; // 这里传根节点为了支持实现 end--，注意下面迭代器也要跟着改动

	RBTreeIterator(Node* node, Node* root)
		:_node(node)
		,_root(root)
	{ }

	Self& operator++()
	{
		if (_node->_right)
		{
			// 右不为空，访问 cur 右子树的最左节点
			Node* min = _node->_right;
			while (min->_left)
			{
				min = min->_left;
			}

			_node = min;
		}
		else
		{
			// 右为空，访问父亲节点（1.cur 在父亲的左边 2.cur 在父亲的右边）
			Node* cur = _node;
			Node* parent = cur->_parent;
			// cur 可能走到根节点，父节点为NULL
			while (parent && cur == parent->_right)
			{
				cur = parent;
				parent = cur->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Self& operator--()
	{
		if (_node == nullptr) // --end()
		{
			// --end(),特殊处理，走到中序最后一个节点，整棵树的最右节点
			Node* max = _root;
			while (max && max->_right)
			{
				max = max->_right;
			}

			_node = max;
		}
		else if (_node->_left)
		{
			// 左子树不为空，中序左子树最后一个
			Node* max = _node->_left;
			while (max->_right)
			{
				max = max->_right;
			}

			_node = max;
		}
		else
		{
			// 右为空，访问父亲节点（1.cur 在父亲的左边 2.cur 在父亲的右边）
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_left)
			{
				cur = parent;
				parent = cur->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Ref operator*()
	{
		return _node->_data;
	}

	Ptr operator->()
	{
		return &_node->_data;
	}

	bool operator==(const Self& s) const
	{
		return s._node == _node;
	}

	bool operator!=(const Self& s) const
	{
		return s._node != _node;
	}
};

/*
* 在原模板的基础上，新添加个模板类似，该模板类型是仿函数
 */
template<class K, class T, class KeyOfT>
class RBTree
{
	typedef RBTreeNode<T> Node;
public:
	// 迭代器
	typedef RBTreeIterator<T, T&, T*> Iterator;
	typedef RBTreeIterator<T, const T&, const T*> ConstIterator;

	Iterator Begin()
	{
		Node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return Iterator(cur, _root);
	}

	// 这里用 End = NULL 有好处
	// 1. 如果 _root = NULL，cur = NULL，End = Begin
	Iterator End()
	{
		return Iterator(nullptr, _root);
	}

	ConstIterator Begin() const
	{
		Node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return ConstIterator(cur, _root);
	}

	ConstIterator End() const
	{
		return ConstIterator(nullptr, _root);
	}

private:
	Node* _root = nullptr;
};
```

>- 中序遍历（左根右）。 
>- ++：右子树最左 / 向上找右祖先。
>- --：左子树最右 / 向上找左祖先。 
>- --end()特殊处理：从 `nullptr` 跳到最右节点。

## 2.4 Myset & Mymap封装
- **set：RBTree（const K防止修改key）**
- **map：RBTree, MapKeyOfT>（const K保护key）**
- **[]运算符：map特有，insert({key, V()}） + 返回second引用**
- **坑点讲解：const后迭代器报错的根源 + 修复方式**
```cpp
// Myset.h
#pragma once

#include"RBTree.h"

namespace yuuki
{
	template<class K>
	class set
	{
		struct SetKeyOfT
		{
			const K& operator()(const K& key)
			{
				return key;
			}
		};

	public:
		/*
		* 这里属于：内部类取内嵌类型，因为这里的内模板没有实例化 set，编译器区别不了
		* 无论 isert 还是 iterator 本质都是调用下层
		*/
		typedef typename RBTree<K, const K, SetKeyOfT>::Iterator iterator;
		typedef typename RBTree<K, const K, SetKeyOfT>::ConstIterator const_iterator;


		iterator begin()
		{
			return _t.Begin();
		}

		iterator end()
		{
			return _t.End();
		}

		const_iterator begin() const
		{
			return _t.Begin();
		}

		const_iterator end() const
		{
			return _t.End();
		}

		pair<iterator, bool> insert(const K& key)
		{
			return _t.Insert(key);
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair
		/*
		* 因为 set 不能修改，这里可以将 value 的值改为const，但是会出现巨大的坑
		* 修改后会显示 _t.End()报错，但这部分没有错误
		* 
		* 原因：因为这里改成 const int,传给红黑树第二个模板参数就是 const int 类型
		* 最终红黑树里 _node->data 节点为 const int 类型，解引用放返回也不能修改，所以 _t.End()报错
		* 只需要将 Myset.h 中的 typedef typename RBTree<K, K, SetKeyOfT>::Iterator iterator 改成 ->
		*					   typedef typename RBTree<K, const K, SetKeyOfT>::Iterator iterator
		*/
		RBTree<K, const K, SetKeyOfT> _t;
	};
}

// Mymap.h
#pragma once

#include"RBTree.h"

namespace yuuki
{
	/*
	* 在 stl_map 中，第二个模板参数用的是 T，改为 V（Value）会更好
	 */
	template<class K, class V>
	class map
	{
		struct MapKeyOfT
		{
			const K& operator()(const pair<K, V>& kv)
			{
				return kv.first;
			}
		};

	public:
		/*
		* 这里属于：内部类取内嵌类型，因为这里的内模板没有实例化 set，编译器区别不了
		* 无论 isert 还是 iterator 本质都是调用下层
		*/
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::Iterator iterator;
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::ConstIterator const_iterator;


		iterator begin()
		{
			return _t.Begin();
		}

		iterator end()
		{
			return _t.End();
		}

		const_iterator begin() const
		{
			return _t.Begin();
		}

		const_iterator end() const
		{
			return _t.End();
		}

		pair<iterator, bool> insert(const pair<K, V>& kv)
		{
			return _t.Insert(kv);
		}

		// C++中，[]充当了插入，查找，修改的功能，需要包 insert
		V& operator[](const K& key)
		{
			pair<iterator, bool> ret = insert({ key, V() });
			return ret.first->second;
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair 
		/*
		* 这里跟 set 一样，把 pair 中第一个模板参数改成 const K，也要记得将上面红黑树的模板参数也一同更改
		*  只需要将 Mymap.h 中的 typedef typename RBTree<K, pair<K, V>, MapKeyOfT>::Iterator iterator; 改成 ->
		*					   typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::Iterator iterator;
		*/
		RBTree<K, pair<const K, V>, MapKeyOfT> _t;
	};
}
```
## 2.5 测试代码 & 输出
```cpp
// Myset.h
#pragma once

#include"RBTree.h"

namespace yuuki
{
	template<class K>
	class set
	{
		struct SetKeyOfT
		{
			const K& operator()(const K& key)
			{
				return key;
			}
		};

	public:
		/*
		* 这里属于：内部类取内嵌类型，因为这里的内模板没有实例化 set，编译器区别不了
		* 无论 isert 还是 iterator 本质都是调用下层
		*/
		typedef typename RBTree<K, const K, SetKeyOfT>::Iterator iterator;
		typedef typename RBTree<K, const K, SetKeyOfT>::ConstIterator const_iterator;


		iterator begin()
		{
			return _t.Begin();
		}

		iterator end()
		{
			return _t.End();
		}

		const_iterator begin() const
		{
			return _t.Begin();
		}

		const_iterator end() const
		{
			return _t.End();
		}

		pair<iterator, bool> insert(const K& key)
		{
			return _t.Insert(key);
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair
		/*
		* 因为 set 不能修改，这里可以将 value 的值改为const，但是会出现巨大的坑
		* 修改后会显示 _t.End()报错，但这部分没有错误
		* 
		* 原因：因为这里改成 const int,传给红黑树第二个模板参数就是 const int 类型
		* 最终红黑树里 _node->data 节点为 const int 类型，解引用放返回也不能修改，所以 _t.End()报错
		* 只需要将 Myset.h 中的 typedef typename RBTree<K, K, SetKeyOfT>::Iterator iterator 改成 ->
		*					   typedef typename RBTree<K, const K, SetKeyOfT>::Iterator iterator
		*/
		RBTree<K, const K, SetKeyOfT> _t;
	};
}

// Mymap.h
#pragma once

#include"RBTree.h"

namespace yuuki
{
	/*
	* 在 stl_map 中，第二个模板参数用的是 T，改为 V（Value）会更好
	 */
	template<class K, class V>
	class map
	{
		struct MapKeyOfT
		{
			const K& operator()(const pair<K, V>& kv)
			{
				return kv.first;
			}
		};

	public:
		/*
		* 这里属于：内部类取内嵌类型，因为这里的内模板没有实例化 set，编译器区别不了
		* 无论 isert 还是 iterator 本质都是调用下层
		*/
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::Iterator iterator;
		typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::ConstIterator const_iterator;


		iterator begin()
		{
			return _t.Begin();
		}

		iterator end()
		{
			return _t.End();
		}

		const_iterator begin() const
		{
			return _t.Begin();
		}

		const_iterator end() const
		{
			return _t.End();
		}

		pair<iterator, bool> insert(const pair<K, V>& kv)
		{
			return _t.Insert(kv);
		}

		// C++中，[]充当了插入，查找，修改的功能，需要包 insert
		V& operator[](const K& key)
		{
			pair<iterator, bool> ret = insert({ key, V() });
			return ret.first->second;
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair 
		/*
		* 这里跟 set 一样，把 pair 中第一个模板参数改成 const K，也要记得将上面红黑树的模板参数也一同更改
		*  只需要将 Mymap.h 中的 typedef typename RBTree<K, pair<K, V>, MapKeyOfT>::Iterator iterator; 改成 ->
		*					   typedef typename RBTree<K, pair<const K, V>, MapKeyOfT>::Iterator iterator;
		*/
		RBTree<K, pair<const K, V>, MapKeyOfT> _t;
	};
}

// RBTree.h
#pragma once
#include <iostream> 
#include <utility>
using namespace std;

enum Colour
{
	RED,
	BLACK
};

/*
* 这里的 RBTree 里的模板参数要跟 _stl_tree 保持一致
* 原本为固定的数据类型 pair<K, V> 改成 泛型 T _data;
 */
template<class T>
struct RBTreeNode
{
	T _data;              // 键值对
	RBTreeNode<T>* _left;     // 左孩子
	RBTreeNode<T>* _right;    // 右孩子
	RBTreeNode<T>* _parent;   // 父节点（红黑树必须维护父节点）
	Colour _col;                 // 节点颜色

	// 构造函数
	RBTreeNode(const T& data)
		:_data(data)
		, _left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _col(RED)// 默认红（插入时默认红，减少黑高破坏）
	{}
};

/*
* 考虑到有 iterator 和 const_iterator 两种迭代器
* 需要传 Ref 和 Ptr 模板参数
 */
template<class T, class Ref, class Ptr>
struct RBTreeIterator
{
	typedef RBTreeNode<T> Node;
	typedef RBTreeIterator<T, Ref, Ptr> Self;

	Node* _node;
	Node* _root; // 这里传根节点为了支持实现 end--，注意下面迭代器也要跟着改动

	RBTreeIterator(Node* node, Node* root)
		:_node(node)
		,_root(root)
	{ }

	Self& operator++()
	{
		if (_node->_right)
		{
			// 右不为空，访问 cur 右子树的最左节点
			Node* min = _node->_right;
			while (min->_left)
			{
				min = min->_left;
			}

			_node = min;
		}
		else
		{
			// 右为空，访问父亲节点（1.cur 在父亲的左边 2.cur 在父亲的右边）
			Node* cur = _node;
			Node* parent = cur->_parent;
			// cur 可能走到根节点，父节点为NULL
			while (parent && cur == parent->_right)
			{
				cur = parent;
				parent = cur->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Self& operator--()
	{
		if (_node == nullptr) // --end()
		{
			// --end(),特殊处理，走到中序最后一个节点，整棵树的最右节点
			Node* max = _root;
			while (max && max->_right)
			{
				max = max->_right;
			}

			_node = max;
		}
		else if (_node->_left)
		{
			// 左子树不为空，中序左子树最后一个
			Node* max = _node->_left;
			while (max->_right)
			{
				max = max->_right;
			}

			_node = max;
		}
		else
		{
			// 右为空，访问父亲节点（1.cur 在父亲的左边 2.cur 在父亲的右边）
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_left)
			{
				cur = parent;
				parent = cur->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Ref operator*()
	{
		return _node->_data;
	}

	Ptr operator->()
	{
		return &_node->_data;
	}

	bool operator==(const Self& s) const
	{
		return s._node == _node;
	}

	bool operator!=(const Self& s) const
	{
		return s._node != _node;
	}
};

/*
* 在原模板的基础上，新添加个模板类似，该模板类型是仿函数
 */
template<class K, class T, class KeyOfT>
class RBTree
{
	typedef RBTreeNode<T> Node;
public:
	// 迭代器
	typedef RBTreeIterator<T, T&, T*> Iterator;
	typedef RBTreeIterator<T, const T&, const T*> ConstIterator;

	Iterator Begin()
	{
		Node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return Iterator(cur, _root);
	}

	// 这里用 End = NULL 有好处
	// 1. 如果 _root = NULL，cur = NULL，End = Begin
	Iterator End()
	{
		return Iterator(nullptr, _root);
	}

	ConstIterator Begin() const
	{
		Node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return ConstIterator(cur, _root);
	}

	ConstIterator End() const
	{
		return ConstIterator(nullptr, _root);
	}

	/*
	* 1. 不支持[]，只返回 bool 即可
	* 
	* 2. 若支持[]，需返回 pair<Iterator, bool>
	*     - 当插入成功，返回 pair<Iterator, true>
	*     - 当插入失败，返回 pair<Iterator, false>
	*/
	pair<Iterator, bool> Insert(const T& data)
	{
		if (_root == nullptr)
		{
			_root = new Node(data);
			_root->_col = BLACK;

			// return ture
			// return pair<Iterator, bool>(Iterator(_root, _root), true);
			return { Iterator(_root, _root), true };
		}

		/*
		* set 直接用 data(key) 比较是没有问题的，但是 map 不能用 pair 进行比较
		* map 的比较要用到 data.first(pair<K, V>._kv.first)，但 set 又不支持用 first
		*
		* 如何解决：需另外用到仿函数(KeyOfT)
		 */
		KeyOfT kot;
		Node* parent = nullptr;
		Node* cur = _root;
		while (cur)
		{
			
			if (kot(cur->_data) < kot(data))
			{
				parent = cur;
				cur = cur->_right;
			}
			else if (kot(cur->_data) > kot(data))
			{
				parent = cur;
				cur = cur->_left;
			}
			else
			{
				return { Iterator(cur, _root), false };
			}
		}

		cur = new Node(data);
		Node* newnode = cur;
		cur->_col = RED;	// 新节点为红色
		if (kot(parent->_data) < kot(data))
			parent->_right = cur;
		else
			parent->_left = cur;
		// 链接父亲
		cur->_parent = parent;

		// 父亲是红色，出现连续的红色节点（需处理）
		while (parent && parent->_col == RED) // 分两种情况：1.叔叔在左边 2.叔叔在右边
		{	// 条件parent：防止空指针（_root节点的父亲为NULL）
			Node* grandfater = parent->_parent;

			if (parent = grandfater->_left) // 叔叔在右边
			{
				//   g
				// p   u
				Node* uncle = grandfater->_right;
				if (uncle && uncle->_col == RED)	// 叔叔存在且为红色（变色）
				{
					parent->_col = uncle->_col = BLACK;
					grandfater->_col = RED;

					// 继续向上处理，最坏结果处理到根
					cur = grandfater;
					parent = cur->_parent;
				}
				else // 叔叔不存在，或存在且为黑（旋转+变色）
				{
					if (cur == parent->_left) // c在父亲左边，构成直线，只单旋一次
					{
						//     g
						//   p   u
						// c
						RotateR(grandfater);
						parent->_col = BLACK;
						grandfater->_col = RED;
					}
					else// c在父亲右边，构成折现，需要双旋
					{
						//     g
						//   p   u
						//    c
						RotateL(parent);
						RotateR(grandfater);
						cur->_col = BLACK;
						grandfater->_col = RED;
					}

					break;
				}
			}
			else // 叔叔在左边（类似上列代码）
			{
				//   g
				// u   p
				Node* uncle = grandfater->_left;

				// 叔叔存在且为红（变色即可）
				if (uncle && uncle->_col == RED)
				{
					parent->_col = uncle->_col = BLACK;
					grandfater->_col = RED;

					// 继续向上处理
					cur = grandfater;
					parent = cur->_parent;
				}
				else// 叔叔不存在，或存在且为黑（旋转+变色）
				{
					//     g
					//   u   p
					//         c
					if (cur == parent->_right)
					{
						RotateL(grandfater);
						parent->_col = BLACK;
						grandfater->_col = RED;
					}
					else
					{
						//     g
						//   u   p
						//      c
						RotateR(parent);
						RotateL(grandfater);
						cur->_col = BLACK;
						grandfater->_col = RED;
					}

					break;
				}
			}
		}
		_root->_col = BLACK; // _root节点必为BLACK

		/*
		* 这里由 { Iterator(cur, _root), true } 变成 
		* { Iterator(newnode, _root), true }; 
		* 这是为了避免因为 [] 而连续变色
		*/
		return { Iterator(newnode, _root), true };

	}

	// 查找接口
	Node* Find(const K& key)
	{
		Node* cur = _root;
		while (cur)
		{
			if (cur->kot(data) < key)
			{
				cur = cur->_right;
			}
			else if (cur->kot(data) > key)
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

	// 获取树的大小
	int Size() { return _Size(_root); }

	// 获取树的高度
	int Height() { return _Height(_root); }

private:
	int _Size(Node* root)
	{
		if (root == nullptr)
			return 0;

		return _Size(root->_left) + _Size(root->_right) + 1;
	}

	int _Height(Node* root)
	{
		if (root == nullptr)
			return 0;
		int leftHeight(root->_left);
		int rightHeight(root->_right);
		return rightHeight > leftHeight ? leftHeight + 1 : rightHeight + 1;
	}

	// 左旋转：以parent为旋转点，右孩子上位
	void RotateL(Node* parent)
	{
		Node* subR = parent->_right;    // 失衡节点的右孩子（新根）
		Node* subRL = subR->_left;      // 新根的左子树（需要转移）
		Node* pParent = parent->_parent; // 失衡节点的父节点

		// 1. 转移subRL：挂到parent的右子树
		parent->_right = subRL;
		if (subRL)
			subRL->_parent = parent;

		// 2. 父节点降级：parent作为subR的左孩子
		subR->_left = parent;
		parent->_parent = subR;

		// 3. 链接新根到原父节点
		if (pParent == nullptr)
		{
			_root = subR;
			subR->_parent = nullptr;
		}
		else
		{
			if (pParent->_left == parent)
				pParent->_left = subR;
			else
				pParent->_right = subR;
			subR->_parent = pParent;
		}
	}

	// 右旋转：以parent为旋转点，左孩子上位
	void RotateR(Node* parent)
	{
		Node* subL = parent->_left;    // 失衡节点的左孩子（新根）
		Node* subLR = subL->_right;    // 新根的右子树（需要转移）
		Node* pParent = parent->_parent; // 失衡节点的父节点

		// 1. 转移subLR：挂到parent的左子树
		parent->_left = subLR;
		if (subLR)
			subLR->_parent = parent;

		// 2. 父节点降级：parent作为subL的右孩子
		subL->_right = parent;
		parent->_parent = subL;

		// 3. 链接新根到原父节点
		if (pParent == nullptr)
		{
			// 原parent是根节点，更新根
			_root = subL;
			subL->_parent = nullptr;
		}
		else
		{
			if (pParent->_left == parent)
				pParent->_left = subL;
			else
				pParent->_right = subL;
			subL->_parent = pParent;
		}
	}

	void Destory(Node* root)
	{
		if (root == nullptr)
			return;

		Destory(root->_left);
		Destory(root->_right);
		delete root;
	}

	Node* Copy(Node* root)
	{
		if (root == nullptr)
			return nullptr;

		Node* newRoot = new Node(root->_kv);
		newRoot->_left = Copy(root->_left);
		newRoot->_right = Copy(root->_right);

		return newRoot;
	}

private:
	Node* _root = nullptr;
};

// test.cpp
#define _CRT_SECURE_NO_WARNINGS
#include"RBTree.h"
#include"Myset.h"
#include"Mymap.h"

// 实现反向迭代器
void set_re_print(const yuuki::set<int>& s)
{
	yuuki::set<int>::const_iterator it = s.end();
	while(it != s.begin())
	{
		--it;
		cout << *it << " ";
	}
	cout << endl;
}

int main()
{
	yuuki::set<int> s;
	s.insert(5);
	s.insert(1);
	s.insert(3);
	s.insert(2);
	s.insert(6);

	yuuki::set<int>::iterator sit = s.begin();
	/*
	* set 不能修改，详见 Myset.h 中 private 中的解析
	*/
	//*sit += 10;
	while (sit != s.end())
	{
		cout << *sit << " ";
		++sit;
	}
	cout << endl;

	for (auto& e : s)
	{
		cout << e << " ";
	}
	cout << endl;

	set_re_print(s);

	yuuki::map<string, string> dict;
	dict.insert({ "sort", "排序" });
	dict.insert({ "left", "左边" });
	dict.insert({ "right", "右边" });

	dict["left"] = "左边，剩余";
	dict["insert"] = "插入";
	dict["string"];


	yuuki::map<string, string>::iterator it = dict.begin();
	while (it != dict.end())
	{
		cout << it->first << ":" << it->second << endl;
		++it;
	}
	cout << endl;

	for (auto& e : dict)
	{
		/*
		* map 中的 frist 不能修改，详见 Mymap.h 中 private 中的解析
		*/
		//e.first += 'x';
		e.second += 'y';
		cout << e.first << ":" << e.second << endl;
	}
	cout << endl;

	return 0;
}
```
# 三、 总结 & 心得
- 红黑树封装set/map的核心是**KeyOfT仿函数**统一key提取 + **const保护key不可变**
- 迭代器实现是难点：中序遍历规则 + --end()边界处理
- 最大的收获：理解了STL容器底层为什么这么设计（平衡性能、接口统一、安全性）
- 下一步：继续卷多线程、C++17/20新特性、LeetCode红黑树相关题。

欢迎评论交流，一起卷C++底层！ 点赞+收藏+关注，不迷路～ヾ(≧▽≦*)o

（完）