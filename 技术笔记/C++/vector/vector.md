>**前言**
>上一篇`模拟实现string`中，讲解了string类的底层是如何实现的，介绍了`string框架、空间增容、三大默认函数、增删查改等功能`
>这篇将往下探索vector(顺序表)的基本用法

---
# 一、为什么要学 vector？它的用处和意义在哪？
在 C++ 中，数组是我们最早接触的容器，但它有一个明显的缺陷：**大小固定**，一旦声明后就无法动态调整容量，当数据量超过数组大小的时，需要手动扩容（比如重新申请更大的空间、拷贝数据、释放旧空间），操作繁琐且容易出错。

而`vector`作为 STL 中的**动态顺序表**，完美解决了数组的痛点：
>**核心优势**：
>1. **动态扩容**：无需手动管理空间，当数据存满时，`vector`会自动申请更大的内存空间（不同编译器扩容策略不同，如 VS 是 1.5 倍、g++ 是 2 倍），并完成数据迁移。
>2. **封装便捷接口**：提供了丰富的成员函数（如`push_back`尾插、`pop_back`尾删、`resize`调整大小等），无需重复编写基础功能代码，提高开发效率。
>3. **连续内存空间**：和数组一样，`vector`的元素在内存中连续存储，支持随机访问（通过`operator[]`或迭代器），访问效率高（时间复杂度 O (1)）。
>4. **适配算法库**：可以直接配合 STL 中的算法（如`find`查找、`sort`排序等），简化复杂操作。

简单来说，`vector`是日常开发中最常用的容器之一，无论是存储一组数据、实现动态数组、还是作为其他复杂数据结构的基础，它都能胜任，是 C++ 开发者必须掌握的工具。
# 二、迭代器中iterator的使用

| 方式            | 适用场景                                                    |
| ------------- | ------------------------------------------------------- |
| begin + end   | 获取首数据位置的iterator， 获取末尾数据的下一个位置的iterator                 |
| rbegin + rend | 获取末尾数据位置的reverse_iterator， 获取首数据的下一个位置的reverse_iterator |
```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<vector>
using namespace std;

int main()
{
    vector<int> v2 = {1, 2, 3, 4, 5}; // 初始化列表（C++11后支持）

    // 正向迭代器：从前往后访问
    vector<int>::iterator it = v2.begin(); // begin()返回首元素迭代器
    while (it != v2.end()) // end()返回尾元素的下一个位置迭代器（哨兵位）
    {
        cout << *it << " "; // 解引用获取元素
        ++it; // 迭代器++移动到下一个元素
    }
    cout << endl; // 输出：1 2 3 4 5

    // 反向迭代器：从后往前访问（rbegin()是最后一个元素，rend()是第一个元素的前一个位置）
    vector<int>::reverse_iterator rit = v2.rbegin();
    while (rit != v2.rend())
    {
        cout << *rit << " ";
        ++rit; // 反向迭代器++是向前移动
    }
    cout << endl; // 输出：5 4 3 2 1

    return 0;
}
```

# 三、常见构造

| 方式                                                        | 适用场景         |
| --------------------------------------------------------- | ------------ |
| vector()                                                  | 无参构造         |
| vector(size_type n, const value_type& val = value_type()) | 构造并初始化n个val  |
| vector(const vector& x)                                   | 拷贝构造         |
| vector(inputlterator first, inputlterator last)           | 使用迭代器进行初始化构造 |
```cpp
#include<vector>
using namespace std;

int main()
{
    vector<int> v1; // 无参构造：创建空vector，size=0，capacity=0（不同编译器可能有差异）

    vector<int> v2(10, 1); // 构造并初始化10个1：size=10，capacity=10
    for (auto num : v2) cout << num << " "; // 范围for遍历（C++11，可补充此遍历方式）
    cout << endl; // 输出：1 1 1 1 1 1 1 1 1 1

    vector<int> v3(++v2.begin(), --v2.end()); // 迭代器区间构造：取v2的[1,9)位置元素
    for (auto num : v3) cout << num << " ";
    cout << endl; // 输出：1 1 1 1 1 1 1 1（共8个1）

    vector<int> v4 = v3; // 拷贝构造：v4是v3的副本
    vector<int> v5{1,2,3,4,5}; // 初始化列表构造（C++11），等价于vector<int> v5 = {1,2,3,4,5}

    return 0;
}
```
# 四、容量问题

| 方式       | 适用场景              |
| -------- | ----------------- |
| size     | 获取数据个数            |
| capacity | 获取容量大小            |
| empty    | 判断是否为空            |
| resize   | 改变vector的size     |
| reserve  | 改变vector的capacity |
>- vs中capacity是按1.5倍增长，g++是按2倍增长
>- reserve只负责开辟空间（当确定需要开多少空间，可以减少扩容所需时间）
>- resize在开空间同时还会进行初始化，影响size
```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<vector>
using namespace std;

int main()
{
    vector<int> v;
    // 观察扩容过程（以g++为例，初始capacity=0，每次翻倍）
    size_t cap = v.capacity();
    for (int i = 0; i < 10; ++i)
    {
        v.push_back(i);
        if (v.capacity() != cap)
        {
            cap = v.capacity();
            cout << "扩容后capacity：" << cap << endl; // 输出：1,2,4,8,16（g++下）
        }
    }

    // reserve：只改容量，不改大小，不初始化新空间
    v.reserve(20);
    cout << "size=" << v.size() << ", capacity=" << v.capacity() << endl; // size=10, capacity=20

    // resize：改大小，可能改容量，会初始化新空间（超出原size的部分）
    v.resize(15, 100); // 大小扩到15，新增的5个元素初始化为100
    cout << "size=" << v.size() << ", capacity=" << v.capacity() << endl; // size=15, capacity=20
    for (int i = 10; i < 15; ++i) cout << v[i] << " "; // 输出：100 100 100 100 100

    v.resize(5); // 大小缩到5，超出部分元素被"舍弃"（不改变容量）
    cout << "\nsize=" << v.size() << ", capacity=" << v.capacity() << endl; // size=5, capacity=20

    return 0;
}
```

# 五、增删查改

| 方式         | 适用场景     |
| ---------- | -------- |
| push_back  | 尾插       |
| pop_back   | 尾删       |
| find       | 查找       |
| insert     | 在指定位置前插入 |
| erase      | 删除指定位置   |
| swap       | 交换数据空间   |
| operator[] | 像数组一样访问  |
```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<vector>
#include<algorithm> // find函数需要包含此头文件
using namespace std;

int main()
{
    vector<int> v;
    // 尾插
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
    for (auto num : v) cout << num << " "; // 1 2 3

    // 尾删
    v.pop_back();
    for (auto num : v) cout << num << " "; // 1 2

    // 查找（find是STL算法，不是vector成员函数）
    auto it = find(v.begin(), v.end(), 2);
    if (it != v.end())
    {
        cout << "\n找到元素：" << *it << endl; // 找到元素：2
    }

    // 在指定位置插入（在it位置前插入10）
    it = v.insert(it, 10); // insert返回新插入元素的迭代器
    for (auto num : v) cout << num << " "; // 1 10 2

    // 删除指定位置元素
    it = v.erase(it); // erase返回删除元素的下一个迭代器
    for (auto num : v) cout << num << " "; // 1 2

    // 交换两个vector的内容（交换底层空间，效率高）
    vector<int> v2{4,5,6};
    v.swap(v2);
    cout << "\nv的元素：";
    for (auto num : v) cout << num << " "; // 4 5 6

    return 0;
}
```

# 六、vector实际运用场景
>vector 的应用非常广泛，举几个常见场景：
>- **存储动态数据**：比如读取未知数量的用户输入，用 vector 动态存储。
>- **作为函数返回值**：返回多个同类型结果（无需关心内存释放）。
>- **实现二维数组**：`vector<vector<int>>`可以动态创建行和列可变的二维数组。
```cpp
// 示例：动态读取数据并排序
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

int main()
{
    vector<int> nums;
    int num;
    cout << "输入多个整数（输入-1结束）：";
    while (cin >> num && num != -1)
    {
        nums.push_back(num);
    }

    // 排序（使用STL算法）
    sort(nums.begin(), nums.end());

    cout << "排序后：";
    for (auto n : nums) cout << n << " "; // 输出排序后的结果
    return 0;
}
```
`vector`作为最基础的动态容器，理解它的原理和用法，对后续学习其他 STL 容器（如 list、map 等）也有很大帮助