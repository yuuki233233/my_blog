>**前言：**
>前面我们也就接触过STL中的部分容器：`string、vector、list、deque、array等`，这些容器统称为序列式容器，因为逻辑结构为线性序列的数据结构，两个位置存储的值之间一般没有紧密的关联关系。
>这篇将带大家来认识关联式容器`map/set系列`和`unordered_map/unordered_set系列`
# 一、序列式容器和关联式容器
顺序容器：元素按存储位置来顺序保存和访问，交换数据不会破坏结构
关联式容器：非线性结构存储，交换数据结构就被破坏
# 二、set系列的使用
## 2.1 set类的介绍
- set的声明如下，T就是set底层关键字的类型
- set默认要求T支持小于比较，如果不支持或者想按自己2的需求走，可以自行实现仿函数传给第二个模板参数
- set底层存储数据的内存是从空间配置器申请的，如果需要可以自己实现内存池，传给第三个参数
- 一般情况下，我们都不需要传后两个模板参数
- set底层是用红黑树实现的，增删查效率是`O(logN)`，迭代器遍历时走的搜索树的中序，所以时有序的
- 前面部分我们已经学习了`vector/list`等容器的使用，STL容器接口设计，高度相似
```cpp
template < class T,
		   class Compare = less<T>,
		   class Alloc = allocatoe<T>
		   > class set;
```
## 2.3 set的构造和恶迭代器
set的构造我们关注以下几个接口即可
set支持正向和反向迭代器遍历，遍历默认按升序排序，因为底层是二叉搜索树，迭代器遍历走的中序。支持迭代器就意味着支持范围for，set的iterator和const_iterator都不支持迭代器修改数据，修改关键字数据就破环了底层搜索树的结构
```cpp
// empty (1) ⽆参默认构造
explicit set (const key_compare& comp = key_compare(),
			  const allocator_type& alloc = allocator_type());
			  
// range (2) 迭代器区间构造
template <class InputIterator>
set (InputIterator first, InputIterator last,
	const key_compare& comp = key_compare(),
	const allocator_type& = allocator_type());

// copy (3) 拷⻉构造
set (const set& x);

// initializer list (5) initializer 列表构造
set (initializer_list<value_type> il,
	const key_compare& comp = key_compare(),
	const allocator_type& alloc = allocator_type());

// 迭代器是⼀个双向迭代器
iterator -> a bidirectional iterator to const value_type

// 正向迭代器
iterator begin();
iterator end();

// 反向迭代器
reverse_iterator rbegin();
reverse_iterator rend();
```
## set的增删查
```cpp
// 单个数据插⼊，如果已经存在则插⼊失败
pair<iterator,bool> insert (const value_type& val);

// 列表插⼊，已经在容器中存在的值不会插⼊
void insert (initializer_list<value_type> il);

// 迭代器区间插⼊，已经在容器中存在的值不会插⼊
template <class InputIterator>
void insert (InputIterator first, InputIterator last);

// 查找val，返回val所在的迭代器，没有找到返回end()
iterator find (const value_type& val);

// 查找val，返回Val的个数
size_type count (const value_type& val) const;

// 删除⼀个迭代器位置的值
iterator erase (const_iterator position);

// 删除val，val不存在返回0，存在返回1
size_type erase (const value_type& val);

// 删除⼀段迭代器区间的值
iterator erase (const_iterator first, const_iterator last);

// 返回⼤于等val位置的迭代器
iterator lower_bound (const value_type& val) const;

// 返回⼤于val位置的迭代器
iterator upper_bound (const value_type& val) const;
```
## 2.3 insert和迭代器遍历
```cpp
void set01() // set插入、打印
{
	// 内置类型
	set<int> s;
	s.insert(1);
	s.insert(2);
	s.insert(3);

	/*for (auto e : s)
		cout << e << " ";*/

	set<int>::iterator it = s.begin();
	while (it != s.end())
	{
		//*it = 1; set不支持修改
		cout << *it << " ";
		it++;
	} // 1 2 3
	cout << endl;

	// C++11写法(可连续插入多个数据)
	s.insert({ 4, 5, 6 });
	for (auto e : s)
		cout << e << " ";
	cout << endl;

	// 外置类型
	set<string> str;
	str.insert({ "hello"," ", "world" });
	for (auto e : str)
		cout << e << " ";
	cout << endl;
}
```
## 2.4 find和erase
```cpp
void set02() // 删除
{
	set<int> s({ 1, 2, 3 });
	for (auto e : s)
		cout << e << " "; // 1 2 3
	cout << endl;

	// erase删除(只支持头删)
	s.erase(s.begin());
	for (auto e : s)
		cout << e << " "; // 2 3
	cout << endl;

	//s.erase(s.end()); // 尾删报错
	//for (auto e : s)
	//	cout << e << " ";
	//cout << endl;

	int era = s.erase(2); // erase删除返回整形
	for (auto e : s)
		cout << e << " "; // 2 3
	cout << endl;
	if (era == 0) // 删除失败返回0
	{
		cout << "删除失败" << endl;
	}
	else // 成功返回非0
	{
		cout << "删除成功" << endl;
	}
}

void set03() // 查找
{
	set<int> s({ 1, 2, 3, 4, 5, 6 });
	for (auto e : s)
	{
		cout << e << " "; // 1 2 3 4 5 6
	}
	cout << endl;

	// 直接查找在利⽤迭代器删除x
	auto e = s.find(3); // 查找到返回迭代器
	if (e != s.end())
	{
		s.erase(e); // 删除迭代器上的数据，后迭代器失效
	}
	for (auto e : s)
	{
		cout << e << " "; // 1 2 4 5 6
	}
	cout << endl;

	int x;
	cin >> x;
	// 利⽤count间接实现快速查找
	if (s.count(x))
	{
		cout << "存在" << endl;
	}
	else
	{
		cout << "不存在" << endl;
	}

	// 算法库的查找 O(N)
	auto pos1 = find(s.begin(), s.end(), x);
	// set⾃⾝实现的查找 O(logN)
	auto pos2 = s.find(x);
}

void set04() // 区间删除
{
	set<int> s({ 1, 2, 3, 4, 5, 6, 7 });
	for (auto e : s)
	{
		cout << e << " ";
	}
	cout << endl;

	// 实现查找到的[itlow,itup)包含[2, 6]区间
	// 返回 >= 2
	auto itlow = s.lower_bound(2);
	// 返回 > 6
	auto itup = s.upper_bound(6);

	// 删除这段区间的值
	s.erase(itlow, itup);
	for (auto e : s)
	{
		cout << e << " "; // 1 6 7
	}
}
```
## 2.7 multiset和set差异
multiset和set的使用基本完全类似，主要区别在于multiset支持数据冗余，与set差异的是insert、find、count、erase
```cpp
void set05() // multiset set
{
	// 相比set不同的是，multiset是排序，但是不去重
	multiset<int> s = { 4,2,7,2,4,8,4,5,4,9 };
	for (auto e : s)
	{
		cout << e << " ";
	}
	cout << endl;

	// 相⽐set不同的是，x可能会存在多个，find查找中序的第⼀个
	int x;
	cin >> x;
	auto pos = s.find(x);
	while (pos != s.end())
	{
		if (*pos == x)
		{
			cout << *pos << " ";
		}
		++pos;
	}
	cout << endl;

	// 相⽐set不同的是，count会返回x的实际个数
	cout << s.count(x) << endl;

	// 相⽐set不同的是，erase给值时会删除所有的x
	s.erase(x);
	for (auto e : s)
	{
		cout << e << " ";
	}
	cout << endl;
}
```
# 三、map系列的使用
## 3.1 map类的介绍
map的声明如下，Key就是map底层关键字的类型，T是map底层value的类型，set默认要求Key支持小于比较，如果不支持或者需要就可以自行实现仿函数传给第二个模板参数，map底层存储数据的内存是从空间配置器申请的。一般情况下，我们不需要传后两个模板参数。map底层也是用红黑树实现，增删查改的效率是`O(logN)`，迭代器遍历时走的中序，所以是按Key有序遍历
```cpp
template < class Key, // map::key_type
		   class T,   // map::mapped_type
		   class Compare = less<Key> // map::key_compare
		   class Alloc = allocator<pair<const Key, T>> // map::allocator_type
		   > class map;
```
## 3.2 pair类型
map底层的红黑树节点中的数据，使用pair<Key, T>存储键值对数据
简单来说：frist -> key，second -> value
```cpp
typedef pair<const Key, T> value_type;

typedef pair<const Key, T> value_type;
struct<class T1, class T2>
{
	typedef T1 first_type;
	typedef T2 second_type;
	
	T1 first;
	T2 second;
	
	pair():first(T1()), second(T2())
	{}
	
	pair(const T1& a, const T2& b):first(a), second(b)
	{}
	
	template<class U, class V>
	pair(const pair<U,V>& pr):first(pr.first), second(pr.second)
	{}
};

template <class T1, class T2>
inline pair<T1, T2> make_pair(T1 x, T2 y)
{
	return (pair<T1,T2>(x,y));
}
```
## 3.3 map插入
### 3.3.1 插入种类
map插入有多种写法，这里就一一例举常见的几种用法
```cpp
#include<map>
#include<iostream>
using namespace std;

int main()
{
	// 内层的{"left", "左边"}隐式类型转换 -> pair，外层的dict = {}隐式类型转换 -> map
	map<string, string> dict = { {"left", "左边"}, {"right", "右边"}, {"insert", "插入"}, {"cout", "输出"} };

	// 方式1：有名对象
	//map<string, string> dict;
	pair<string, string> kv1("first", "第一个");
	dict.insert(kv1);

	// 方式2：匿名对象
	dict.insert(pair<string, string>("second", "第二个"));

	// 方式3：调用make_pair(函数模板，编译器自动推理模板类型并返回)
	dict.insert(make_pair("sort", "排序"));

	// 方式4：隐式类型转换
	dict.insert({ "auto", "自动" });

	// 插入是只看key，value不相等不会更新
	dict.insert({ "auto", "zidong" });

	// 支持迭代器
	map<string, string>::iterator it = dict.begin();
	while (it != dict.end())
	{
		// 可以修改value，不支持修改key
		//it->first += 'x';
		it->second += 'x';

		// C++不支持返回多个值
		//cout << *it << endl;

		//cout << (*it).first << ":" << (*it).second << endl;
		cout << it->first << ":" << it->second << endl;
		// it->first访问是省略后的，下面是原始的访问
		//cout << it.operator->()->first << ":" << it.operator->()->second << endl;
		++it;
	}
	return 0;
}
```
### 3.3.2 pair中的key和value
pair存在于map中，类似构成一个节点。而pair中
