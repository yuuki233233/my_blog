# 一、auto 关键字：简化类型声明的利器
>- **自动类型推导**：无需显式指定类型，编译器根据初始化值确定变量类型
>- **简化复杂声明**：对于迭代器、函数返回值等冗长类型，`auto` 能大幅精简代码
>- **指针与引用规则**：
>	- 声明指针时，`auto` 与 `auto*` 效果一致（如 `auto p = &a` 与 `auto* p = &a` 等价）
>	- 声明引用必须显式添加 `&`（如 `auto& ref = a`）
>- **多变量声明限制**：同一行声明的多个变量必须为相同类型（如 `auto a = 1, b = 2` 合法，`auto a = 1, b = 2.0` 非法）
>- **使用限制**：
>	- 必须在声明时初始化（无法单独声明 `auto x;`）
>	- 不能作为函数参数类型（如 `void func(auto x)` 不合法）
>	- 不建议作为函数返回值类型（可能导致类型模糊）
>	- 不能用于声明数组（如 `auto arr[10]` 不合法）

```cpp
#include <iostream>
#include <typeinfo> // 需包含此头文件以使用 typeid
using namespace std;

int getNum() { return 42; }

int main() {
    int a = 10;
    auto b = a;          // 推导为 int
    auto c = 'a';        // 推导为 char
    auto d = getNum();   // 推导为 int
    auto* p = &a;        // 推导为 int*（与 auto p = &a 等价）
    auto& ref = a;       // 推导为 int&

    // 打印推导的类型（不同编译器输出可能略有差异）
    cout << "b 的类型：" << typeid(b).name() << endl; // int
    cout << "c 的类型：" << typeid(c).name() << endl; // char
    cout << "d 的类型：" << typeid(d).name() << endl; // int
    cout << "p 的类型：" << typeid(p).name() << endl; // int*

    return 0;
}
```
# 二、string 类：更安全的字符串处理方案
C++ 标准库的 `string` 类封装了字符串的创建、修改和操作，相比 C 风格字符数组（`char*`），提供了自动内存管理和丰富的成员函数，是字符串处理的首选。
> **核心优势**
>- **自动内存管理**：无需手动 `malloc/free` 或 `new/delete`，避免内存泄漏
>- **安全性**：内置越界检查，减少缓冲区溢出风险
>- **丰富接口**：提供拼接、查找、截取等便捷操作
>- **兼容性**：支持与 C 风格字符串（`const char*`）相互转换
>- **动态扩展**：可自动调整容量以适应字符串长度变化
## 1、常见构造

| constructor函数名称          | 功能说明                                 |
| ------------------------ | ------------------------------------ |
| string()                 | 构造空字符串（默认构造）                         |
| string(const char* s)    | 用 C 风格字符串初始化                         |
| string(const string&s)   | 拷贝构造（复制已有 string 对象）                 |
| string(size_t n, char c) | 构造包含 n 个字符 c 的字符串构造包含n个字符c的string类对象 |
```cpp
#include <string>
using namespace std;

int main() {
    string s1;                  // 空字符串
    string s2("hello");         // 用 C 字符串初始化
    string s3(s2);              // 拷贝构造
    string s4(5, '!');          // 5个感叹号："!!!!!"

    cout << "s2: " << s2 << endl; // 输出：hello
    cout << "s4: " << s4 << endl; // 输出：!!!!!
    return 0;
}
```
## 2、容量操作

| 函数名称                                                                  | 功能说明                         |
| --------------------------------------------------------------------- | ---------------------------- |
| [size](https://cplusplus.com/reference/string/string/size/)           | 返回有效字符长度（推荐使用，替代 `length()`） |
| [empty](https://cplusplus.com/reference/string/string/empty/)         | 判断字符串是否为空（为空返回 `true`）       |
| [clear](https://cplusplus.com/reference/string/string/clear/)         | 清空有效字符（不释放底层内存）              |
| [reserve](https://cplusplus.com/reference/string/string/reserve/)     | 预留至少 n 个字符的空间（提升插入效率）        |
| [resize(n, c)](https://cplusplus.com/reference/string/string/resize/) | 调整有效字符数为 n，多余位置用 c 填充        |
>1. 一般情况下都是用`size()`，而不是`length()`
>2.`clear()`只是将string中有效字符清空，不改变底层空间大小
>3.`resize(n)` 与 `resize(n, c)` 的区别：
    - 前者将多余位置初始化为空字符（'\0'）
    - 后者将多余位置初始化为指定字符 c
    - 若n大于当前容量，会扩容；若n小于当前长度，仅截断不缩容
>4.`reserve(n)` 仅预留空间，不改变有效字符数，n 小于当前容量时不做操作

```cpp
#include<iostream>
using namespace std;

int main()
{
	string s1("hello world");
	cout << s1.size() << endl;  //11
	cout << s1.empty() << endl; //0
	s1.clear();
	cout << s1 << endl; //打印空

	string s2;
	s2.reserve(100);
	cout << s2.capacity() << endl; //111

	//左(要初始化个数)：右(初始化的字符)
	s2.resize(4, 'c');
	cout << s2 << endl; //cccc

	return 0;
}
```
## 3、访问及遍历操作

| 方式                                                                                                                          | 适用场景                                                       |
| --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [operator[]](https://cplusplus.com/reference/string/string/operator[]/)                                                     | 通过下标访问（支持读写，类似数组）                                          |
| [begin](https://cplusplus.com/reference/string/string/begin/)+[end](https://cplusplus.com/reference/string/string/end/)     | 正向迭代器，从首到尾遍历（适用于所有 STL 容器），`begin()`指向首字符，`end()` 指向尾字符后一位 |
| [rbegin](https://cplusplus.com/reference/string/string/rbegin/)+[rend](https://cplusplus.com/reference/string/string/rend/) | 反向迭代器，从尾到首遍历，`rbegin()`指向尾字符，`rend()`指向首字符前一位              |
| 范围for                                                                                                                       | C++11 新增，简洁遍历所有元素（底层基于迭代器）                                 |
```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string s("hello");

    // 1. 下标访问
    cout << "第2个字符：" << s[1] << endl; // 'e'
    s[0] = 'H'; // 修改首字符为'H'
    cout << "修改后：" << s << endl; // Hello

    // 2. 正向迭代器
    cout << "正向遍历：";
    for (auto it = s.begin(); it != s.end(); ++it) {
        cout << *it << " "; // H e l l o
    }
    cout << endl;

    // 3. 反向迭代器
    cout << "反向遍历：";
    for (auto it = s.rbegin(); it != s.rend(); ++it) {
        cout << *it << " "; // o l l e H
    }
    cout << endl;

    // 4. 范围 for（C++11）
    cout << "范围for遍历：";
    for (char ch : s) {
        cout << ch << " "; // H e l l o
    }
    cout << endl;

    return 0;
}
```

## 4、string 类的其他常用操作

| 函数名称             | 功能说明                                            |
| ---------------- | ----------------------------------------------- |
| operator+=       | 字符串拼接（支持 string 或 C 字符串）                        |
| append()         | 尾部追加字符串（功能类似 `+=`）                              |
| c_str()          | 返回 C 风格字符串（`const char*`）                       |
| find(sub, pos)   | 从 pos 位置开始查找子串 sub，返回起始索引（未找到返回 `string::npos`） |
| substr(pos, len) | 从 pos 位置截取长度为 len 的子串（默认截取到末尾）                  |
| compare()        | 比较与字符串 s 的大小（返回 0 表示相等）                         |

```cpp
#include<iostream>
#include<string>
using namespace std;

int main()
{
    string s1 = "hello";
    string s2 = "world";
    
    // 字符串拼接
    s1 += " ";
    s1 += s2;
    cout << s1 << endl; // 输出：hello world
    
    // 查找操作
    size_t pos = s1.find("world");
    if (pos != string::npos)
    {
        cout << "找到子串，位置：" << pos << endl; // 输出：6
    }
    
    // 截取子串
    string s3 = s1.substr(6, 5);
    cout << s3 << endl; // 输出：world
    
    // C风格字符串转换
    const char* cstr = s1.c_str();
    cout << cstr << endl; // 输出：hello world
    
    return 0;
}
```
# 三、常用遍历
## 1、C++11遍历
适用于数组和支持下标访问的容器（如 string、vector），需要手动控制索引范围。
```cpp
#include<iostream>
#include<string>
using namespace std;

int main()
{
	//C++98遍历
	int array1[] = { 1,2,3,4,5 };
	for (int i = 0; i < sizeof(array1) / sizeof(array1[0]); ++i)
	{
		array1[i] *= 2;
	}
	for (int i = 0; i < sizeof(array1) / sizeof(array1[0]); ++i)
	{
		cout << array1[i] << " ";
	}
	cout << endl;

	return 0;
}
```
## 2、迭代器遍历
迭代器是 STL 容器的通用遍历方式，适用于所有容器`（包括不支持下标访问的容器如 list、map 等）`。
![](图片/图片/QQ20251203-213626.png)
```cpp
#include<iostream>
#include<string>
using namespace std;

int main()
{
	string s1("hello world");

	//正向迭代器
	//string::iterator it = s1.begin();
	auto it = s1.begin();
	while (it != s1.end())
	{
		cout << *it << " ";
		++it;
	}
	cout << endl;
	
	//反向迭代器
	//string::reverse_iterator rit = s1.rbegin();
	auto rit = s1.rbegin();
	while (rit != s1.rend())
	{
		cout << *rit << " ";
		++rit;
	}
	cout << endl;

	const string s2("hello world");
	//const正向迭代器
	//string::const_iterator cit = s2.begin();
	auto cit = s2.begin();
	while (cit != s2.end())
	{
		cout << *cit << " ";
		++cit;
	}
	cout << endl;

	//const反向迭代器
	//string::const_reverse_iterator rcit = s2.rbegin();
	auto rcit = s2.rbegin();
	while (rcit != s2.rend())
	{
		cout << *rcit << " ";
		++rcit;
	}

	return 0;
}
```
## 3、for遍历
一种简洁的遍历方式，自动迭代容器中所有元素，底层基于迭代器实现。语法格式：
```cpp
for (元素类型 变量名 : 容器名) {
    // 循环体
}
```

```cpp
#include<iostream>
#include<string>
using namespace std;

int main()
{
    // 数组遍历
    int array2[] = { 1,2,3,4,5 };
    for (auto& e : array2)  // 使用引用避免拷贝，支持修改元素
        e *= 2;

    for (auto e : array2)
        cout << e << " ";   // 输出：2 4 6 8 10
    cout << endl;

    // 字符串遍历
    string str("hello world");
    for (auto ch : str)
    {
        cout << ch << " ";  // 输出：h e l l o  w o r l d
    }
    cout << endl;

    return 0;
}
```
# 四、string 类的实现原理（进阶）
了解 string 类的实现有助于更好地理解其特性，下面展示两种经典的实现方式：
## 传统写法
通过显式分配和释放内存实现深拷贝，确保每个对象拥有独立的字符串资源：
```cpp
class String
{
public:
    // 构造函数
    String(const char* str = "")
    {
        if (nullptr == str)
        {
            assert(false);
            return;
        }
        _str = new char[strlen(str) + 1];  // 分配空间（包含结束符）
        strcpy(_str, str);                 // 拷贝内容
    }
    
    // 拷贝构造函数
    String(const String& s)
        : _str(new char[strlen(s._str) + 1])
    {
        strcpy(_str, s._str);
    }
    
    // 赋值运算符重载
    String& operator=(const String& s)
    {
        if (this != &s)  // 避免自赋值
        {
            char* pStr = new char[strlen(s._str) + 1];
            strcpy(pStr, s._str);
            delete[] _str;  // 释放旧空间
            _str = pStr;    // 指向新空间
        }
        return *this;
    }
    
    // 析构函数
    ~String()
    {
        if (_str)
        {
            delete[] _str;
            _str = nullptr;
        }
    }

private:
    char* _str;  // 存储字符串
};
```
## 现代写法
通过交换临时对象的资源简化代码，利用临时对象的生命周期自动释放内存
```cpp
class String
{
public:
    // 构造函数
    String(const char* str = "")
    {
        if (nullptr == str)
        {
            assert(false);
            return;
        }
        _str = new char[strlen(str) + 1];
        strcpy(_str, str);
    }
    
    // 拷贝构造函数（现代写法）
    String(const String& s)
        : _str(nullptr)
    {
        String strTmp(s._str);  // 创建临时对象
        swap(_str, strTmp._str); // 交换资源
    }

    // 赋值运算符重载（现代写法）
    String& operator=(String s)  // 传值参数会触发拷贝构造
    {
        swap(_str, s._str);      // 交换资源，临时对象会自动释放旧资源
        return *this;
    }
    
    // 析构函数
    ~String()
    {
        if (_str)
        {
            delete[] _str;
            _str = nullptr;
        }
    }

private:
    char* _str;
};
```
## 五、string 类的实际应用场景
>1. **文本处理**：日志记录、配置文件解析、字符串格式化
>2. **用户交互**：命令行输入输出、GUI 文本控件
>3. **网络编程**：HTTP 协议处理、数据报文组装与解析
>4. **文件操作**：路径处理、文件内容读写
>5. **数据转换**：数值与字符串的相互转换
>掌握 `auto` 和 `string` 是 C++ 开发的基础，合理使用能显著提升代码简洁性和安全性。实际开发中，建议优先使用标准库提供的 `string` 而非 C 风格字符串，减少内存管理风险。