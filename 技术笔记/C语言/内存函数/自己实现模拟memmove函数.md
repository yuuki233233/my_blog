# 我是如何通过画图理解并实现 memmove 的
---

## 遇到的难题
---
原来以为内存拷贝很简单，直到遇到重叠内存的情况

## 画图分析过程
---
![](图片/图片/QQ20251005-225425.png)
- 发现如果从前往后拷贝会覆盖数据
- 理解为什么需要从后往前拷贝

## 我的实现代码
---
![[图片/图片/屏幕截图 2025-10-05 222251.png]]
---
```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<string.h>

//不希望src被修改，前面加const
void* my_memmove(void* dest,const void* src, int sz)
{
	void* s = src;
	dest = (char*)dest + sz - 1;
	src = (char*)src + sz - 1;
	while (sz--)
	{
		*(char*)dest = *(char*)src;
		dest = (char*)dest - 1;
		src = (char*)src - 1;
	}
	return s;
}

int main()
{
	int arr[] = { 1,2,3,4,5,6,7,8,9,10 };
	int len = sizeof(arr) / sizeof(arr[0]);
	void* set = my_memmove(arr + 3, arr, 5 * sizeof(int));

	for (int i = 0; i < len; i++)
	{
		printf("%d ", arr[i]);
	}
	return 0;
}
```

# 学到的教训

1. 画图是理解指针和内存的最佳工具
2. 考虑边界情况很重要
3. 逆向思维在编程中很常用

# 我的代码亮点：
---
1. 参数设计很专业：使用const void* src保护源数据
2. 类型转化正确：(char*)转换现实字节操作
3. 逆向拷贝思路：从后往前复刻，避免重叠覆盖
4. 返回值设置：返回源指针，与标准库一致

# 改进建议
---
实现是从后往前拷贝，这在 dest > src 的情况下是正确的。但完整的 memmove 应该根据内存重叠情况选择方向：

```c
void* my_memmove(void* dest, const void* src, size_t sz) 
{
    char* d = (char*)dest;
    const char* s = (const char*)src;
    
    if (d < s) {
        // 从前往后拷贝
        for (size_t i = 0; i < sz; i++)
            d[i] = s[i];
    } else if (d > s) {
        // 从后往前拷贝
        for (size_t i = sz; i > 0; i--)
            d[i-1] = s[i-1];
    }
    return (void*)dest;
}
```