# 从零实现 C++ list 类：手把手拆解双向循环链表的底层逻辑

> **前言**
> 
> 在上一篇中，我们吃透了 vector 的底层实现 —— 作为动态连续数组，它凭借 “随机访问” 的优势成为日常开发的首选，但也存在无法回避的短板：头部 / 中间插入删除需要挪动大量元素，时间复杂度高达 O (n)；扩容时的内存拷贝也会带来额外性能开销。
> 
> 而 list 作为 STL 中另一核心容器，恰好弥补了 vector 的这些不足：它基于**双向循环链表**实现，任意位置的插入删除仅需修改指针指向，时间复杂度可降至 O (1)。本章我们将从链表的底层结构出发，一步步实现一个功能完整的 list 类，带你掌握：
> 
> 1. 双向循环链表的设计逻辑与核心优势
> 2. list 与 vector 的底层差异及适用场景
> 3. 链表迭代器的特殊实现（为什么不能直接用指针？）

## 一、list 容器的核心特性

`list` 是 STL 中以双向循环链表为底层结构的序列式容器，其核心特性包括：

1. **非连续存储**：元素在内存中离散分布，通过指针连接形成链表
2. **双向遍历**：每个节点包含前驱和后继指针，支持向前 / 向后遍历
3. **高效增删**：任意位置的插入 / 删除操作仅需修改指针指向，时间复杂度为 O (1)
4. **无扩容开销**：无需预先分配内存，元素增减不会导致大规模内存拷贝
5. **迭代器特殊**：迭代器不是原生指针，需重载 `++`/`--` 等运算符实现节点跳转

**与 vector 的核心差异**：

| 特性     | vector           | list           |
| ------ | ---------------- | -------------- |
| 存储方式   | 连续内存空间           | 离散链表节点         |
| 随机访问   | 支持（O (1)）        | 不支持（需遍历）       |
| 插入删除效率 | 中间插入删除效率低（O (n)） | 任意位置效率高（O (1)） |
| 内存开销   | 小（仅存储数据）         | 大（需额外存储指针）     |
| 扩容机制   | 自动扩容（可能有拷贝）      | 无扩容机制          |

# 二、list 的迭代器使用
`list` 的迭代器是实现链表遍历的关键，由于其非连续存储特性，迭代器的实现与 `vector` 有本质区别。

| 方式                    | 适用场景                                     |
| --------------------- | ---------------------------------------- |
| `begin()` + `end()`   | 正向迭代器，`begin()` 指向首元素，`end()` 指向尾元素下一位r  |
| `rbegin()` + `rend()` | 反向迭代器，`rbegin()` 指向尾元素，`rend()` 指向首元素前一位 |

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> l = {1, 2, 3, 4, 5};
    
    // 正向迭代器遍历
    cout << "正向遍历: ";
    for (list<int>::iterator it = l.begin(); it != l.end(); ++it) {
        cout << *it << " ";
    }
    cout << endl;  // 输出：1 2 3 4 5
    
    // 反向迭代器遍历
    cout << "反向遍历: ";
    for (list<int>::reverse_iterator rit = l.rbegin(); rit != l.rend(); ++rit) {
        cout << *rit << " ";
    }
    cout << endl;  // 输出：5 4 3 2 1
	
    return 0;
}
```
**注意**：`list` 的迭代器不支持随机访问（如 `it + 3` 操作），只能通过 `++`/`--` 逐步移动。

# 三、list 的常见构造方式
`list` 提供了多种构造函数，满足不同场景下的初始化需求：

| 方式                                            | 适用场景                     |
| --------------------------------------------- | ------------------------ |
| list()                                        | 无参构造，创建空链表               |
| list(size_type n, const T& val = T())         | 构造包含 n 个 val 元素的链表       |
| list(const list& x)                           | 拷贝构造，创建 x 的副本            |
| list(InputIterator first, InputIterator last) | 用 [first, last) 区间元素构造链表 |
| list(initializer_list< T > ilist)             | 初始化列表构造（C++11）           |
```cpp
#include <iostream>
#include <list>
using namespace std;

void printList(const list<int>& l) {
    for (auto num : l) {
        cout << num << " ";
    }
    cout << endl;
}

int main() {
    // 无参构造
    list<int> l1;
    
    // 构造包含5个3的链表
    list<int> l2(5, 3);
    printList(l2);  // 输出：3 3 3 3 3
    
    // 迭代器区间构造
    list<int> l3(l2.begin(), --l2.end());
    printList(l3);  // 输出：3 3 3 3
    
    // 拷贝构造
    list<int> l4(l3);
    printList(l4);  // 输出：3 3 3 3
    
    // 初始化列表构造（C++11）
    list<int> l5{1, 2, 3, 4, 5};
    printList(l5);  // 输出：1 2 3 4 5
    
    return 0;
}
```

# 四、list 的容量与元素访问

`list` 提供了基础的容量查询和元素访问接口：

| 方式         | 适用场景                    |
| ---------- | ----------------------- |
| empty()    | 判断链表是否为空，为空返回 true      |
| size()     | 返回链表中有效元素的个数            |
| front()    | 返回链表第一个元素的引用            |
| back()     | 返回链表最后一个元素的引用           |
| max_size() | 返回链表理论上能容纳的最大元素个数（很少使用） |
```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<int> l = {10, 20, 30, 40, 50};
    
    cout << "链表是否为空: " << (l.empty() ? "是" : "否") << endl;  // 输出：否
    cout << "链表元素个数: " << l.size() << endl;  // 输出：5
    cout << "第一个元素: " << l.front() << endl;  // 输出：10
    cout << "最后一个元素: " << l.back() << endl;  // 输出：50
    
    // 修改首尾元素
    l.front() = 100;
    l.back() = 500;
    for (auto num : l) {
        cout << num << " ";
    }
    // 输出：100 20 30 40 500
    
    return 0;
}
```
**注意**：`list` 不支持 `operator[]` 下标访问和随机访问，只能通过迭代器或 `front()`/`back()` 访问元素。
# 五、list 的增删查改操作
`list` 提供了丰富的元素操作接口，尤其擅长插入和删除操作：

| 函数声明                                                          | 接口说明                              |
| ------------------------------------------------------------- | --------------------------------- |
| push_front(const T& val)                                      | 在链表头部插入元素 val                     |
| pop_front()                                                   | 删除链表头部元素                          |
| push_back(const T& val)                                       | 在链表尾部插入元素 val                     |
| pop_back()                                                    | 删除链表尾部元素                          |
| insert(iterator pos, const T& val)                            | 在 pos 位置前插入元素 val                 |
| insert(iterator pos, size_type n, const T& val)               | 在 pos 位置前插入 n 个 val               |
| insert(iterator pos, InputIterator first, InputIterator last) | 在 pos 位置前插入 [first, last) 区间元素    |
| erase(iterator pos)                                           | 删除 pos 位置的元素，返回下一个元素的迭代器          |
| erase(iterator first, iterator last)                          | 删除 [first, last) 区间元素，返回下一个元素的迭代器 |
| swap(list& x)                                                 | 交换当前链表与 x 中的元素                    |
| clear()                                                       | 清空链表中的所有元素                        |
| remove(const T& val)                                          | 删除链表中所有值为 val 的元素                 |
| unique()                                                      | 删除连续的重复元素（只保留一个）                  |
| sort()                                                        | 对链表元素进行排序（升序）                     |
| reverse()                                                     | 反转链表元素的顺序                         |
```cpp
#include <iostream>
#include <list>
#include <algorithm> // 用于find算法
using namespace std;

void printList(const list<int>& l, const string& msg) {
    cout << msg << ": ";
    for (auto num : l) {
        cout << num << " ";
    }
    cout << endl;
}

int main() {
    list<int> l;
    
    // 尾插元素
    l.push_back(1);
    l.push_back(2);
    l.push_back(3);
    printList(l, "尾插后");  // 输出：1 2 3
    
    // 头插元素
    l.push_front(0);
    printList(l, "头插后");  // 输出：0 1 2 3
    
    // 查找元素（使用STL算法）
    auto it = find(l.begin(), l.end(), 2);
    if (it != l.end()) {
        // 在找到的位置前插入元素
        l.insert(it, 100);
        printList(l, "插入后");  // 输出：0 1 100 2 3
    }
    
    // 删除元素
    it = find(l.begin(), l.end(), 1);
    if (it != l.end()) {
        l.erase(it);
        printList(l, "删除后");  // 输出：0 100 2 3
    }
    
    // 排序
    l.sort();
    printList(l, "排序后");  // 输出：0 2 3 100
    
    // 反转
    l.reverse();
    printList(l, "反转后");  // 输出：100 3 2 0
    
    // 移除指定值元素
    l.remove(3);
    printList(l, "移除3后");  // 输出：100 2 0
    
    // 清空链表
    l.clear();
    cout << "清空后是否为空: " << (l.empty() ? "是" : "否") << endl;  // 输出：是
    
    return 0;
}
```
>**注意**：
>1. `list` 自带 `sort()` 成员函数，不建议使用 STL 中的 `sort` 算法（效率低）
>2. 插入操作不会使迭代器失效，删除操作只会使被删除元素的迭代器失效
>3. `unique()` 仅删除连续的重复元素，通常需配合 `sort()` 使用以删除所有重复元素
# 六、list的实际运用
>`list` 凭借其高效的插入删除特性，适用于以下场景：
>1. **频繁插入删除的场景**：如实现队列、栈、双向队列等数据结构
>2. **数据元素较大的场景**：避免 `vector` 扩容时的大量数据拷贝
>3. **需要频繁在两端操作的场景**：如实现 LRU 缓存淘汰算法

# 七、总结
`list` 作为基于双向循环链表的容器，在元素插入删除操作上具有显著优势，但不支持随机访问。使用时需根据具体场景选择：
- 需频繁随机访问数据 → 选择 `vector`
- 需频繁插入删除数据 → 选择 `list`
- 数据量小且访问模式不确定 → 可优先考虑 `vector`（简单高效）

掌握 `list` 的迭代器特性和成员函数用法，能帮助我们在合适的场景下写出更高效的代码。在实际开发中，合理搭配不同容器的优势，才能发挥 STL 的最大威力。