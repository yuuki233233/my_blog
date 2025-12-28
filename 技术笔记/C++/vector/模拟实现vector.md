# 从零实现 C++ string 类：手把手教你造轮子
# 模拟实现vector
>**前言**
>在上一章节中讲解了STL库里面vector的用法，其中有`构造、迭代器访问、运算符重载、插入等接口`，想更近一步了解vector，就要从底层的代码开始，一步一步造出vector的轮子
>本章你将掌握
>1. vector的底层实现
>2. 迭代器失效原理

# 一、vector的基本框架设计
其实vector类似于string类，在本质上都是一个顺序表，其接口大致一致。vector中的成员是以指针的形式表示的，如`_start (指向数组开头的指针)、_finish(指向数组末尾的指针)、_end_of_storage(指向总空间的末尾)
```cpp
#include<iostream>
#include<assert.h>
using namespace std;

namespace yuuki
{
    // 模板支持不同类型的vector<int>、vector<double>等
    template<class T>
    class vector
    {
    public:
        typedef T* iterator;         // 可修改元素的迭代器
        typedef const T* const_iterator;  // 只读迭代器
    
    private:
        iterator _start;  // 指向数组起始位置
        iterator _finish; // 指向有效元素的末尾
        iterator _end_of_storage; // 指向总容量的末尾
    };
}
```
这里使用我的博客名`yuuki`作为命名空间是为了避免与标准库的vector冲突。STL库里的vector使用迭代器的模板方式，是为了更好地支持多种数据类型，提升库的通用性。
# 二、基础功能实现
## 2.1 迭代器支持
为了支持范围 for 循环、元素遍历及扩容等功能，需要实现迭代器相关接口：
```cpp
#include<iostream>
#include<assert.h>
using namespace std;

namespace yuuki
{
	template<class T>
	class vector
	{
	public:
		typedef T* iterator;
		typedef const T* const_iterator;
		
		// 迭代器功能实现
		iterator begin(){return _start;}
		const_iterator being(){return _start;} // 只读版本
		iterator end(){return _finish;}
		const_iterator end(){return _finish;}  // 只读版本
		
		// 打印vector元素(模板函数支持任意类型)
		template<class T>
		void print_vector(const vector<T>& v)
		{
			// 规定，没有实例化的类模板里面取东西，编译器不能区分这里const_iterator
			// 是类型还是静态成员变量
			//typename vector<T>::const_iterator it = v.begin();
			auto it = v.begin();
		
			while (it != v.end())
			{
				cout << *it << " ";
				++it;
			}
			cout << endl;
		}
		
	private:
	iterator _start = nullptr;
	iterator _finish = nullptr;
	iterator _end_of_storage = nullptr;
};
```
## 2.2 常用访问接口
提供获取 vector 状态信息的接口：
```cpp
// 获取有效元素个数
size_t size() const
{
    return _finish - _start;
}

// 获取总容量大小
size_t capacity() const
{
    return _end_of_storage - _start;
}

// 判断vector是否为空
bool empty() const
{
    return _finish == _start;
}
```
# 三、默认构造
## 3.1构造函数
```cpp
/*vector()		
{}*/

// C++11 前置生成默认构造
vector() = default;
```
## 3.2拷贝构造
```cpp
// v2(*this) = v1
vector(const vector<T>& v)
{
	reserve(size());
	for (auto& e : v)
	{
		push_back(e);
	}
}
```
## 3.3析构函数
```cpp
~vector()
{
	if (_start)
	{
		delete[] _start;
		_start = _finish = _end_of_storage = nullptr;
	}
}
```
# 四、内存管理：reserve与resize
## 4.1扩容函数 reserve
想要实现vector类，就要先了解作为工具接口reserve(扩容)
```cpp
void reserve(size_t n)
{
    if (n > capacity())  // 仅当需要的空间大于当前容量时扩容
    {
        size_t old_size = size();  // 保存旧的有效元素个数
        T* tmp = new T[n];         // 开辟新空间
        
        // 拷贝旧元素到新空间（注意：此处简化实现，未考虑自定义类型的拷贝构造）
        if (_start)  // 避免空指针访问
        {
            memcpy(tmp, _start, sizeof(T) * old_size);
            delete[] _start;  // 释放旧空间
        }
        
        // 更新指针指向
        _start = tmp;
        _finish = tmp + old_size;
        _end_of_storage = tmp + n;
    }
}
```
## 4.2调整大小函数 resize
resize 可在调整元素个数的同时初始化新元素，也可截断超出的元素：
```cpp
// 三种情况：n小于当前size（截断）、n大于当前size但不超过capacity（补值）、n超过capacity（扩容+补值）
void resize(size_t n, T val = T())
{
    if (n < size())
    {
        _finish = _start + n;  // 直接截断
    }
    else
    {
        reserve(n);  // 确保容量足够
        // 填充新元素
        while (_finish < _start + n)
        {
            *_finish = val;
            ++_finish;
        }
    }
}
```
# 五、运算符重载
## 5.1元素访问下标
vector 底层为连续空间，支持下标访问：
```cpp
// 可修改元素的版本
T& operator[](size_t pos)
{
    assert(pos < size());  // 检查下标合法性
    return _start[pos];
}

// 只读版本
const T& operator[](size_t pos) const
{
    assert(pos < size());  // 检查下标合法性
    return _start[pos];
}
```
## 5.2赋值运算符
```cpp
// v2 = v1 (现代写法)
void swap(vector<T>& v)
{
	std::swap(_start, v._start);
	std::swap(_finish, v._finish);
	std::swap(_end_of_storage, v._end_of_storage);
}

vector<T> operator=(vector<T> v)
{
	swap(v);
	return *this;
}
```

# 六、插入与删除操作
## 6.1 插入元素
包括尾插和指定位置插入：
```cpp
// 尾插元素
void push_back(const T& ch)
{
    // 容量不足时扩容（初始容量为0则先扩至4）
    if (_finish == _end_of_storage)
    {
        reserve(capacity() == 0 ? 4 : capacity() * 2);
    }

    *_finish = ch;  // 插入元素
    ++_finish;      // 更新有效元素末尾
}

// 在指定位置插入元素
iterator insert(iterator pos, const T& x)
{
    // 检查迭代器合法性
    assert(pos >= _start && pos <= _finish);
    
    // 容量不足时扩容，并修正pos（避免迭代器失效）
    if (_finish == _end_of_storage)
    {
        size_t len = pos - _start;  // 记录pos与起始位置的偏移量
        reserve(capacity() == 0 ? 4 : capacity() * 2);
        pos = _start + len;  // 重新定位pos
    }

    // 元素后移
    iterator end = _finish - 1;
    while (end >= pos)
    {
        *(end + 1) = *end;
        --end;
    }
    
    *pos = x;  // 插入新元素
    ++_finish; // 更新有效元素末尾
    
    return pos;  // 返回新插入元素的位置
}
```
## 6.2 删除元素
包括尾删和指定位置删除：
```cpp
// 尾删元素
void pop_back()
{
    assert(!empty());  // 确保vector非空
    --_finish;
}

// 删除指定位置元素
iterator erase(iterator pos)
{
    // 检查迭代器合法性
    assert(pos >= _start && pos < _finish);
    
    // 元素前移
    iterator it = pos + 1;
    while (it < _finish)
    {
        *(it - 1) = *it;
        ++it;
    }
    
    --_finish;  // 更新有效元素末尾
    return pos;  // 返回删除位置的下一个元素
}
```
# 七、迭代器失效问题深度解析
迭代器失效本质是迭代器指向的底层空间被释放，或指向的位置已不再合法。
## 7.1 底层空间改变导致失效
涉及空间重新分配的操作（如`resize、reserve、insert、push_back、assign`等）可能导致旧空间被释放，原迭代器失效：
```cpp
int main()
{
    vector<int> v{1,2,3,4,5,6};
    auto it = v.begin();
    
    v.reserve(100);  // 可能触发扩容，旧空间释放
    // 此时it指向已释放的旧空间，访问会导致未定义行为
    while(it != v.end())
    {
        cout << *it << " ";  // 崩溃风险
        ++it;
    }
    return 0;
}
```
**解决办法**：操作后重新获取迭代器（如`it = v.begin()`）。
## 7.2 删除操作导致失效
`erase`删除元素后，被删除位置的迭代器及后续迭代器可能失效：
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    vector<int> v{1,2,3,4};
    auto pos = find(v.begin(), v.end(), 3);
    
    v.erase(pos);
    cout << *pos << endl;  // pos已失效，访问非法
    return 0;
}
```
**解决办法**：利用`erase`的返回值更新迭代器（如`it = v.erase(it)`）。
# 八、测试用例验证
为确保实现正确性，编写测试用例：
```cpp
void test_vector()
{
    // 测试int类型vector
    vector<int> v;
    v.push_back(1);
    v.push_back(2);
    v.print_vector(v);  // 预期输出：1 2
    
    // 测试插入功能
    v.insert(v.begin(), 30);
    v.insert(v.begin() + 2, 20);
    v.print_vector(v);  // 预期输出：30 1 20 2
    
    // 测试删除功能（删除偶数）
    auto it = v.begin();
    while (it != v.end())
    {
        if (*it % 2 == 0)
        {
            it = v.erase(it);  // 用返回值更新迭代器
        }
        else
        {
            ++it;
        }
    }
    v.print_vector(v);  // 预期输出：30 1
    
    // 测试double类型vector
    vector<double> vd;
    vd.push_back(1.1);
    vd.push_back(2.2);
    vd.print_vector(vd);  // 预期输出：1.1 2.2
}
```
# 九、总结与思考
通过手动实现 vector，我们完成了从 “使用 STL 容器” 到 “理解容器本质” 的跨越：
>1. **核心原理**：vector 是 “动态连续数组”，通过三个指针管理内存，扩容的本质是 “重新开辟空间 + 拷贝数据 + 释放旧空间”，扩容策略（2 倍扩容）是时间 / 空间效率的平衡；
>2. **关键避坑**：深拷贝是容器实现的核心，浅拷贝会导致内存重复释放；迭代器失效的根源是 “指针指向的空间 / 位置非法”，需通过重新获取迭代器或利用返回值更新迭代器解决；
>3. **细节优化**：原代码中`memcpy`仅支持简单类型，逐元素赋值可兼容自定义类型，这也是 STL 源码中常用的兼容方案；
>4. **拓展方向**：本次实现为基础版本，STL 原生 vector 还包含移动语义（C++11）、emplace 系列接口（直接构造元素，减少拷贝）、allocator 内存分配器等高级特性，后续可逐步补充。

“造轮子” 的意义从来不是重复造工具，而是通过拆解和重构，搞懂 “工具为什么这么设计”。当你能解释清 “为什么拷贝构造要深拷贝”“为什么 insert 要返回新迭代器”，就真正掌握了 vector 的设计思想 —— 这种底层思维，会让你在使用 STL 时更有底气，遇到问题也能快速定位根源。

后续我们会继续拆解 list（链表）、map（红黑树）等容器的实现，从 “顺序存储” 到 “链式存储” 再到 “树形存储”，一步步打通 STL 的核心脉络。