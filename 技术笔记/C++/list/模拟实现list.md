## 为什么要学 list？

### 1. 补齐容器认知：从 “连续存储” 到 “链式存储”

学习 STL 的核心是理解 “不同存储结构适配不同场景”：

- vector 是 “连续空间” 的代表，优势是随机访问（[] 下标）、缓存命中率高；
- list 是 “离散空间” 的代表，优势是任意位置增删高效、无扩容开销。
    
    只有同时掌握这两种核心容器，才能在实际开发中根据需求选择最优方案 —— 比如存储高频增删的动态数据（如订单列表、消息队列）时，list 的效率远高于 vector。

### 2. 突破迭代器认知：理解 “智能迭代器” 的设计

vector 的迭代器本质是原生指针（T*），但 list 的迭代器无法直接用指针实现：链表节点离散存储，指针 ++/-- 无法定位到下一个 / 上一个节点。

通过实现 list 的迭代器，你会理解 “迭代器是对底层指针的封装” 这一核心思想，掌握自定义迭代器的设计逻辑 —— 这是打通 STL 容器迭代器体系的关键一步。

### 3. 夯实底层编程能力：吃透链表的核心操作

双向循环链表是 C++ 底层编程的经典模型：

- 掌握 “头节点（哨兵位）” 的设计技巧，避免空指针判断的冗余代码；
- 理解 “节点插入 / 删除” 的指针指向逻辑，规避链表操作中常见的野指针、断链问题；
- 对比 list 与 vector 的深拷贝实现差异，深化对 “内存管理” 的理解。

### 4. 工程实践价值：精准选择容器提升性能

实际开发中选错容器，可能导致性能数量级的差距：

- 若用 vector 存储需要频繁中间插入的数据，每次插入都要挪动元素，数据量越大效率越低；
- 若用 list 存储需要频繁随机访问的数据，每次访问都要遍历链表，效率远低于 vector。
    
    学透 list 的底层逻辑，能让你精准判断 “什么时候该用 list，什么时候该用 vector”，而非盲目依赖 vector。
# 一、list整体框架
>list 是基于**双向循环链表**实现的容器，核心组件包括：
>
>1. **节点结构**：存储数据、前驱指针、后继指针
>2. **迭代器**：封装节点指针，实现类似指针的遍历操作`（++/--/*/->）`
>3. **链表类**：管理节点（头节点 / 哨兵位）、维护大小，提供增删查改接口
## 1.1节点(list_node）
list不同于vector，前者是用链表(内存上不连续)，后者是顺序表(内存上是连续的)，因此list在框架上是以一个一个节点构成。
双向链表的节点需要存储数据和两个指针（前驱 + 后继），用 `struct` 定义方便访问成员：
```cpp
template<class T>
struct list_node {
    T _data;          // 节点数据
    list_node* _prev; // 前驱节点指针
    list_node* _next; // 后继节点指针

    // 构造函数：用匿名对象默认初始化（支持无参构造）
    list_node(const T& data = T()) 
        : _data(data), _prev(nullptr), _next(nullptr) {}
};
```
## 1.2迭代器实现（list_iterator）
list 迭代器不能直接用原生指针（节点不连续），需封装节点指针并重载运算符：
关键设计：
- 用模板参数 `Ref`（引用）和 `Ptr`（指针）区分普通迭代器和 const 迭代器
- 重载 `++`/`--` 实现遍历，`*`/`->` 实现数据访问
```cpp
// T(类型)  Ref(迭代器(包含cons迭代器))  Ptr(结构体(两变量以上))
template<class T, class Ref, class Ptr>
struct list_iterator // 要经常访问迭代器，这里用struct
{
	typedef list_node<T> Node; // 节点
	typedef list_iterator<T, Ref, Ptr> Self; // 模板
};
```

## 1.3链表
链表是list的重要组成部分，需要作为类类型单独列出来
```cpp
template<class T>
class list // 链表
{
	typedef list_node Node<T>; // 链表由节点构成，需要调用节点
public:
	typedef list_iterator<T, T&, T*> iterator; // 普通迭代器(可读可写)
	typedef const_list_iterator<T, const T&, const T*> const_iterator; // const迭代器(只读)

private:
	Node* _head;	// 头节点(哨兵位)
	size_t _size;	// 节点数量(避免每次计算大小)
};
```
在VS中_size是不存在的，但为了方便，我们拿来去作计数，是为了方便以后调用大小时不需要访问函数。

# 二、实现迭代器功能
## 2.1构造
在后置++和后置--中，要返回自身的旧迭代器的位置，这就需要实现一个拷贝构造函数来对旧迭代器位置的保留
```cpp
list_iterator(Node* node) // 构造节点
	:_node(node)
{ }
```
## 2.2遍历
遍历主要实现迭代器位置上的变换
```cpp
Node* _node; // 封装的节点指针

// 前置++：返回更新后的自身
Self& operator++()
{
	_node = _node->_next;	// 下一个节点的迭代器
	return *this;			// 返回自身
}

// 后置++：返回更新前的副本
Self operator++(int)
{
	Self tmp(*this);	// 拷贝构造旧迭代器位置
	_node = _node->_next;
	return tmp;
}

// 前置--
Self& operator--()
{
	_node = _node->_prev;	// 上一个节点的迭代器
	return _node;			// 返回自身
}

// 后置--
Self operator--(int)
{
	Self tmp(*this);	// 拷贝构造旧迭代器位置
	_node = _node->_prev;
	return tmp;
}

// 两个迭代器比较是否指向同一节点
bool operator!=(const Self& s)
{
	return _node != s._node;
}

bool operator==(const Self& s)
{
	return _node == s._node;
}
```

## 2.3访问
访问分为节点访问和结构体访问，这需要我们实现两种访问的方式
```cpp
// 解引用：返回数据的引用(支持读写或只读)
Ref operator*()
{
	return _node->_data;
}

// 箭头运算符：返回数据的指针(用于结构体/类成员访问)
Ptr operator->()
{
	return &_node->_data;
}
```

# 三、实现链表功能
## 3.1构造
### 1.构造函数
```cpp
// 构造
list()
{
	_head = new Node; // 给哨兵位提供个节点
	_head->_next = _head;
	_head->_prev = _head; // 节点指向自己
	_size = 0;
}
```
### 2.拷贝构造
```cpp
// 初始化哨兵位(私有工具函数)
void empty_init()
{
	_head = new Node; // 给哨兵位提供个节点
	_head->_next = _head;
	_head->_prev = _head; // 节点指向自己
	_size = 0;
}

// 拷贝构造 lt2(lt1)
list(const list<T>& lt)
{
	empty_init(); // 先初始化自己的哨兵位
	
	// 遍历lt，将元素尾插到当前链表
	for (auto& e : lt) // 尾插出个一样的链表
	{
		push_back(e);
	}
}
```
### 3.赋值(现代写法)
```cpp
// 交换两个链表的资源
void swap(list<T>& lt) {
    std::swap(_head, lt._head);
    std::swap(_size, lt._size);
}

// 赋值：lt2 = lt1（利用拷贝构造+交换实现深拷贝）
list<T>& operator=(list<T> lt) { // 传值参数会拷贝一份lt1
    swap(lt); // 交换当前对象与拷贝的临时对象
    return *this; // 临时对象销毁时释放原资源
}
```
### 4.析构函数
```cpp
// 清理所有有效节点（保留哨兵位）
void clear() {
    iterator it = begin();
    while (it != end()) {
        it = erase(it); // erase返回下一个迭代器
    }
}

// 析构：清理节点+释放哨兵位
~list() {
    clear();
    delete _head;
    _head = nullptr;
}
```
## 3.2 常用接口
```cpp
iterator begin()
{
	/*iterator it(_head->_next); // 有名对象
	return it;*/

	//return iterator(_head->_next); // 匿名对象

	return _head->_next; // 隐式类型转换(第一个有效节点)
}

const_iterator begin() const
{
	return _head->_next;
}

iterator end()
{
	return _head; // 哨兵位(尾节点)
}

const_iterator end() const
{
	return _head;
}

size_t size() const
{
	return _size;
}

// 判空
bool empty() const
{
	// return _head->next == _head;
	return _size == 0;
}
```
## 3.3查找删除
list在元素的插入和删除上有着巨大优势
### 1.插入
再指定位置前插入新节点(迭代器不失效)：
```cpp
// 在pos位置前插入x
iterator insert(iterator pos, const T& x) {
    Node* cur = pos._node;       // 当前位置节点
    Node* prev = cur->_prev;     // 前一个节点
    Node* new_node = new Node(x); // 新节点

    // 调整指针指向
    prev->_next = new_node;
    new_node->_prev = prev;
    new_node->_next = cur;
    cur->_prev = new_node;

    _size++;
    return iterator(new_node); // 返回新插入节点的迭代器
}

// 尾插（调用insert在end()前插入）
void push_back(const T& x) {
    insert(end(), x);
}

// 头插（调用insert在begin()前插入）
void push_front(const T& x) {
    insert(begin(), x);
}
```
### 2.删除
```cpp
// 删除pos位置节点，返回下一个迭代器
iterator erase(iterator pos) {
    assert(pos != end()); // 不能删除哨兵位

    Node* cur = pos._node;
    Node* prev = cur->_prev;
    Node* next = cur->_next;

    // 调整指针跳过当前节点
    prev->_next = next;
    next->_prev = prev;
    delete cur; // 释放节点内存

    _size--;
    return iterator(next); // 返回下一个有效迭代器
}

// 头删（删除begin()位置）
void pop_front() {
    erase(begin());
}

// 尾删（删除end()前一个位置）
void pop_back() {
    erase(--end()); // end()是哨兵位，--后是最后一个有效节点
}
```
# 四、打印链表
```cpp
// 支持所有容器
// 按需实例化
template<class Container>
void print_container(const Container& v)
{
	// const iterator -> 迭代器本身不能修改
	// const_iterator -> 指向内容不能修改

	//list<int>::const_iterator it = v.const_begin();
	auto it = v.begin();
	while (it != v.begin())
	{
		// const迭代器只读不写
		//*it += 10; 
		cout << *it << " ";
		++it;
	}

	for (auto e : v)
	{
		cout << e << " ";
	}
	cout << endl;
}
```
# 五、易错点与补充说明
1. **哨兵位的作用**：
    头节点（哨兵位）不存储数据，自身形成循环（`_prev` 和 `_next` 都指向自己），可避免对空链表的特殊判断（如插入第一个节点时无需检查 `_head` 是否为 `nullptr`）。
    
2. **迭代器失效问题**：
    - `insert` 不会导致迭代器失效（新节点插入后，原迭代器仍指向原节点）。
    - `erase` 会导致当前迭代器失效（节点已被释放），需用其返回值更新迭代器。
3. **const 迭代器的实现**：
    通过模板参数 `Ref=const T&` 和 `Ptr=const T*`，复用同一迭代器结构，避免代码冗余。

**与vector的对比**

| 操作        | vector（连续空间） | list（链表）     |
| --------- | ------------ | ------------ |
| 随机访问      | O (1)（支持 []） | O (n)（需遍历）   |
| 中间插入 / 删除 | O (n)（挪动元素）  | O (1)（仅调整指针） |
| 扩容        | 可能触发（拷贝旧数据）  | 无扩容（按需分配节点）  |

# 六、测试用例验证
为确保实现正确性，编写测试用例：
```cpp
void test_list()
{
	list<int> lt1;
	lt1.push_back(1);
	lt1.push_back(4);
	print_container(lt1); // 输出：1 4

	lt1.insert(++lt1.begin(), 2);
	print_container(lt1); // 输出：1 2 4

	lt1.pop_front();
	lt1.pop_back();
	print_container(lt1); // 输出：2

	// insert以后迭代器不失效
	list<int>::iterator it = lt1.begin();
	lt1.insert(it, 20);
	*it += 100;			// 输出：20 102(迭代器没有失效)
	print_container(lt1);

	lt1.push_back(3);
	// erase以后迭代器失效
	// 删除所有的偶数
	it = lt1.begin();
	while (it != lt1.end())
	{
		if (*it % 2 == 0)
		{
			//lt.erase(it); // 迭代器失效
			it = lt1.erase(it);
		}
		else
		{
			++it;
		}
	}
	print_container(lt1); // 输出：3


	list<int> lt2(lt1);

	print_container(lt1); // 输出：3
	print_container(lt2); // 输出：3


	list<int> lt3;
	lt3.push_back(10);
	lt3.push_back(20);
	lt3.push_back(30);
	lt3.push_back(40);

	lt1 = lt3; // 默认生成浅拷贝
	print_container(lt1); // 输出：10 20 30 40
	print_container(lt3); // 输出：10 20 30 40
}
```
# 七、总结与思考
1. 通过手动实现 list，我们跳出了 “只会调用 STL 接口” 的黑盒使用阶段，真正理解了：
>- **链表的本质**：双向循环链表通过 “哨兵位头节点” 简化边界处理，节点间的指针关联是增删高效>的关键；
>- **迭代器的设计哲学**：迭代器并非只能是原生指针，而是 “对指针行为的封装”—— 通过重载`++`/`--`/`*`/`->`，让离散存储的链表也能像数组一样用统一的迭代器接口遍历；
>- **容器设计的权衡**：list 的 “无扩容、增删 O (1)” 优势与 “随机访问 O (n)” 劣势，本质是 “空间连续性” 与 “操作灵活性” 的取舍。

 2. 易错点复盘：避坑指南
>- **迭代器失效问题**：`erase`会导致当前迭代器指向的节点被释放，必须用返回值更新迭代器；而`insert`不会失效，因为仅新增节点不影响原有节点的指针关联。
>- **拷贝构造的细节**：忘记初始化自己的哨兵位，直接拷贝原链表的节点指针，会导致两个链表共用节点（浅拷贝），析构时双重释放。
>- **指针指向逻辑**：插入 / 删除节点时，需严格按照 “先处理新节点与前驱的关系，再处理新节点与后继的关系” 的顺序调整指针，否则易出现断链或野指针。

3. 我们实现的 list 是简化版本，STL 的`std::list`还有更多工程细节：
>- **内存池**：STL 通过`allocator`管理节点内存，减少频繁`new/delete`的开销；
>- **迭代器分类**：`std::list`的迭代器属于双向迭代器（BidirectionalIterator），不支持`+=n`等随机访问操作，这与 vector 的随机访问迭代器形成鲜明对比；
>- **更多接口**：如`splice`（拼接链表）、`merge`（合并有序链表）等，利用链表特性实现高效操作。

4. 编程思维提炼：抽象与复用
>- **模板的妙用**：通过`Ref`和`Ptr`模板参数，用一套迭代器代码同时实现普通迭代器和 const 迭代器，避免代码冗余；
>- **封装的意义**：将节点操作、迭代器行为封装在类内部，对外暴露简洁接口（如`push_back`/`erase`），使用者无需关心底层指针细节；
>- **对比学习法**：通过与 vector 的对比（存储结构、迭代器类型、性能特性），更深刻理解 “数据结构决定算法效率” 的本质。

通过这个过程，我们不仅掌握了 list 的实现，更重要的是学会了 “透过接口看底层” 的思维 —— 这正是理解复杂框架和库的核心能力。后续可以尝试实现`list`的反向迭代器，或对比`forward_list`（单向链表）的设计差异，进一步深化对链表容器的理解。