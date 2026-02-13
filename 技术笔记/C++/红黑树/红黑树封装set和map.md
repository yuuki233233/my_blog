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
## 2.1 实现出复用红黑树的框架（支持insert）
- 在 stl_map 中，第二个模板参数用的是 T，改为 V（Value）会更好
- set 直接用`data(key)` 比较是没有问题的，但是 map 不能用 pair 进行比较，map 的比较要用到 `data.first(pair<K, V>._kv.first)`，但 set 又不支持用 first。如何解决：需另外用到仿函数(KeyOfT)
```cpp
// Mayset.h
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
		bool Insert(const K& key)
		{
			return _t.Insert(key);
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair 
		RBTree<K, K, SetKeyOfT> _t;
	};
}

// Mymap.h
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
		bool Insert(const V& kv)
		{
			return _t.Insert(kv);
		}

	private:
		// 对于 set 而言，第二个模板参数传的是 key
		// 对于 map 而言，第二个模板参数传的是 pair 
		RBTree<K, pair<K, V>, MapKeyOfT> _t;
	};
}

// RBTree.h
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
   
	// 插入接口
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

		return true;
	}
	
	private:
	Node* _root = nullptr;
};
```
## 2.2 支持iterator迭代器
**iterator实现思路分析：**
- list 和 RBTree 本质上是各个节点链接而成，但不同的是链接的方式不同。list：{节点 -> 节点}，RBTree：{节点} -> {节点1，节点2}
- iterator 实现的框架于 list 的 iterator 思路类似，用一个类型封装节点的指针，在通过重载运算符实现，迭代器像指针一样访问
- 
```cpp

```























