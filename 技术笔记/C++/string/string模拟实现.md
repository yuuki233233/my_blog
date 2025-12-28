# 从零实现 C++ string 类：手把手教你造轮子
# 模拟实现string类
>**前言**
>作为 C++ 程序员，string 类是我们日常开发中最常用的容器之一。但你是否真正了解它的底层实现？今天我们就来一步步模拟实现一个功能完善的 string 类，从基础框架到各种成员函数，带你深入理解字符串的存储与操作原理。
## 一、string 类的基本框架设计
首先，我们需要确定 string 类的核心成员变量。一个字符串类至少需要存储字符数据、当前长度和容量，所以我们定义三个私有成员：
```cpp
#pragma once
#include<iostream>
#include<assert.h>
using namespace std;
 
```cpp
namespace yuuki
{
	class string
	{
	public:
		// 后续会在这里实现各种成员函数
	private:
		char* _str;      // 存储字符串数据
		int _size;       // 当前字符串长度（不包含'\0'）
		int _capacity;   // 容量（最大可存储的字符数，不包含'\0'）
		static const size_t npos;  // 静态常量，用于表示"未找到"
	};
}
```
这里使用我的博客名`yuuki`作为命名空间是为了避免与标准库的 string 冲突。`npos`通常定义为`-1`，由于是 `size_t` 类型，它会被自动转换为该类型的最大值。
## 二、内存管理：扩容函数 reserve
字符串操作经常需要扩容，我们先实现 `reserve` 函数，用于预留存储空间：
```cpp
void reserve(size_t n);

void string::reserve(size_t n)
{
	if (n > _capacity)  // 只有当需要的空间大于当前容量时才扩容
	{
		char* tmp = new char[n + 1];  // +1是为了存放'\0'
		strcpy(tmp, _str);            // 拷贝原有数据
		delete[] _str;                // 释放旧空间
		_str = tmp;                   // 指向新空间
		_capacity = n;                // 更新容量
	}
}
```
**注意**：reserve 只负责扩容，不改变字符串的实际内容和长度。
## 三、默认成员函数实现
任何类都离不开三大默认成员函数：构造函数、拷贝构造函数和析构函数。
### 3.1 构造函数
我们可以将无参构造和带参构造合并，使用缺省参数：
```cpp
// 方法1：分为两函数
//string()
//	:_str(new char[1]{'\0'})
//	,_size(0)
//	,_capacity(0)
//{ }

//string(const char* str)
//{
//	_size = strlen(str);
//	_capacity = _size;
//	// _capacity不包含\0
//	_str = new char[_capacity + 1];
//	strcpy(_str, str);
//}

// 方法2：合并成一个（缺省参数）
string(const char* str = "")
{
	_size = strlen(str);
	_capacity = _size;
	_str = new char[_capacity + 1];  // 预留'\0'的位置
	strcpy(_str, str);
}
```
### 3.2 拷贝构造函数
拷贝构造必须实现深拷贝，否则会导致两个对象共用同一块内存，析构时出现 `double free` 错误：
```cpp
string(const string& str)
{
	// 为新对象分配独立的内存空间
	_str = new char[str._capacity + 1];
	strcpy(_str, str._str);  // 拷贝字符串内容
	_size = str._size;       // 拷贝长度
	_capacity = str._capacity;  // 拷贝容量
}
```
### 3.3 析构函数
析构函数负责释放动态分配的内存：
```cpp
~string()
{
	delete[] _str;    // 释放字符串数组
	_str = nullptr;   // 避免野指针（原代码是==，这里修正为=）
	_size = _capacity = 0;  // 重置成员变量
}
```
## 四、基础功能实现
### 4.1 迭代器支持
为了支持范围 for 循环和标准库算法，我们需要实现迭代器：
```cpp
// 迭代器类型定义
typedef char* iterator;
typedef const char* const_iterator;

// 迭代器接口
iterator begin() { return _str; }
const_iterator begin() const { return _str; }
iterator end() { return _str + _size; }
const_iterator end() const { return _str + _size; }
```
### 4.2 常用访问接口
提供一些获取字符串信息的接口：
```cpp
const char* c_str() const { return _str; }  // 返回C风格字符串
void clear() { _str[0] = '\0'; _size = 0; }  // 清空字符串
size_t size() const { return _size; }        // 获取长度
size_t capacity() const { return _capacity; }  // 获取容量
```
## 五、运算符重载
运算符重载是 string 类的核心功能，让字符串操作更加直观。
### 5.1 赋值运算符
同样需要实现深拷贝：
```cpp
string& operator=(const string& str)
{
	// 避免自我赋值
	if (this != &str)
	{
		delete[] _str;  // 释放原有空间
		_str = new char[str._capacity + 1];
		strcpy(_str, str._str);
		_size = str._size;
		_capacity = str._capacity;
	}
	return *this;  // 支持链式赋值
}
```
### 5.2 下标访问运算符
实现类似数组的访问方式：
```cpp
// 非const版本，支持修改
char& operator[](size_t pos)
{
	assert(pos < _size);  // 越界检查
	return _str[pos];
}

// const版本，仅支持读取
const char& operator[](size_t pos) const
{
	assert(pos < _size);  // 越界检查
	return _str[pos];
}
```
### 5.3 字符串拼接运算符
实现`+=`运算符，支持字符和字符串：
```cpp
// 追加字符
void push_back(char ch)
{
	if (_size == _capacity)
	{
		// 容量为0时先扩到4，否则翻倍
		reserve(_capacity == 0 ? 4 : _capacity * 2);
	}
	_str[_size] = ch;
	++_size;
	_str[_size] = '\0';  // 保持字符串结束符
}

// 追加字符串
void append(const char* str)
{
	size_t len = strlen(str);
	if (_size + len > _capacity)
	{
		// 按需扩容，取更大的值
		reserve(_size + len > 2 * _capacity ? _size + len : 2 * _capacity);
	}
	strcpy(_str + _size, str);  // 直接拷贝到尾部
	_size += len;
}

// 运算符重载
string& operator+=(char ch) { push_back(ch); return *this; }
string& operator+=(const char* str) { append(str); return *this; }
```
### 5.4 关系运算符
实现字符串比较功能：
```cpp
// 小于运算符
bool operator<(const string& s1, const string& s2)
{
	return strcmp(s1.c_str(), s2.c_str()) < 0;
}

// 等于运算符
bool operator==(const string& s1, const string& s2)
{
	return strcmp(s1.c_str(), s2.c_str()) == 0;
}

// 其他运算符可以通过复用上面的实现
bool operator<=(const string& s1, const string& s2) { return s1 < s2 || s1 == s2; }
bool operator>(const string& s1, const string& s2) { return !(s1 <= s2); }
bool operator>=(const string& s1, const string& s2) { return !(s1 < s2); }
bool operator!=(const string& s1, const string& s2) { return !(s1 == s2); }
```
## 六、插入与删除操作
### 6.1 插入操作
实现在指定位置插入字符或字符串：
```cpp
// 插入字符
void insert(size_t pos, char ch)
{
	assert(pos <= _size);  // 允许在末尾插入

	if (_size == _capacity)
	{
		reserve(_capacity == 0 ? 4 : _capacity * 2);
	}

	// 从后往前挪动数据
	size_t end = _size + 1;
	while (end > pos)
	{
		_str[end] = _str[end - 1];
		--end;
	}

	_str[pos] = ch;
	++_size;
}

// 插入字符串
void insert(size_t pos, const char* str)
{
	assert(pos <= _size);

	size_t len = strlen(str);
	if (_size + len > _capacity)
	{
		reserve(_size + len > 2 * _capacity ? _size + len : 2 * _capacity);
	}

	// 挪动数据腾出空间
	size_t end = _size + len;
	while (end > pos + len - 1)
	{
		_str[end] = _str[end - len];
		--end;
	}

	// 拷贝插入字符串
	for (size_t i = 0; i < len; ++i)
	{
		_str[pos + i] = str[i];
	}
	_size += len;  // 更新size
}
```
### 6.2 删除操作
实现从指定位置删除指定长度的字符：
```cpp
void erase(size_t pos, size_t len = npos)
{
	assert(pos < _size);

	if (len >= _size - pos)
	{
		// 删除到末尾
		_str[pos] = '\0';
		_size = pos;
	}
	else
	{
		// 向前挪动数据覆盖要删除的部分
		for (size_t i = pos; i < _size - len; ++i)
		{
			_str[i] = _str[i + len];
		}
		_size -= len;
		_str[_size] = '\0';  // 添加结束符
	}
}
```
## 七、查找与子串
### 7.1 查找功能
实现查找字符和子串的位置：
```cpp
// 查找字符
size_t find(char ch, size_t pos = 0)
{
	for (size_t i = pos; i < _size; ++i)  // 修正原代码：从pos开始查找
	{
		if (_str[i] == ch)
		{
			return i;
		}
	}
	return npos;  // 未找到
}

// 查找子串
size_t find(const char* str, size_t pos = 0)  // 修正原代码：参数类型应为const char*
{
	assert(pos < _size);

	const char* tmp = strstr(_str + pos, str);
	if (tmp == nullptr)
	{
		return npos;
	}
	else
	{
		return tmp - _str;  // 计算相对起始位置
	}
}
```
### 7.2 子串提取
实现从指定位置提取子串：
```cpp
string substr(size_t pos = 0, size_t len = npos)
{
	assert(pos < _size);

	// 处理长度超出剩余部分的情况
	if (len > _size - pos)
	{
		len = _size - pos;
	}

	string sub;
	sub.reserve(len);  // 预留空间，提高效率
	for (size_t i = 0; i < len; ++i)
	{
		sub += _str[pos + i];
	}
	return sub;
}
```
## 八、输入输出运算符
重载流运算符，支持直接输入输出 string 对象：
```cpp
// 输出运算符
ostream& operator<<(ostream& out, const string& s)
{
	// 利用迭代器遍历输出
	for (auto x : s)
	{
		out << x;
	}
	return out;
}

// 输入运算符
istream& operator>>(istream& in, string& s)
{
	s.clear();  // 先清空原有内容

	char ch;
	ch = in.get();  // 使用get()可以读取空格

	// 读取直到遇到空格或换行（'\0'不是输入终止符）
	while (ch != ' ' && ch != '\n')
	{
		s += ch;
		ch = in.get();
	}

	return in;
}
```
## 九、测试用例与验证
为了确保我们实现的 string 类正确无误，编写一些测试用例：
```cpp
void test_string1()
{
	string s1;
	string s2("hello world");
	cout << "s1: " << s1.c_str() << endl;
	cout << "s2: " << s2.c_str() << endl;

	// 测试迭代器
	for (auto e : s2)
	{
		cout << e << " ";
	}
	cout << endl;
}

void test_string2()
{
	string s1("hello");
	s1 += ' ';
	s1 += "world";
	cout << "s1 after append: " << s1 << endl;

	s1.insert(5, ",");
	cout << "s1 after insert: " << s1 << endl;

	s1.erase(5, 1);
	cout << "s1 after erase: " << s1 << endl;

	size_t pos = s1.find("world");
	if (pos != string::npos)
	{
		cout << "found 'world' at position: " << pos << endl;
	}
}
```
## 十、总结与思考
>通过这次模拟实现，我们深入理解了 string 类的底层工作原理：
>1. 字符串本质上是动态分配的字符数组，需要手动管理内存
>2. 深拷贝是实现字符串类的关键，避免内存问题
>3. 合理的扩容策略可以提高字符串操作的效率
>4. 迭代器的实现让 string 类能够兼容 STL 算法
>
当然，标准库的 string 实现更加复杂和高效，还包含了很多优化（如 SSO 短字符串优化等），但核心原理与我们实现的版本是相通的。
>希望通过这篇文章，你能对 C++ string 类有更清晰的认识，在今后的开发中更加得心应手！
