
# 一、复杂度的概念
- 一个算法的好坏，主要是对比两者的**时间和空间**两个维度，也就是时间和空间复杂度。
- 时间复杂度主要衡量一个算法运行的快慢，空间复杂度主要衡量一个算法运行需要的额外空间
# 二、时间复杂度
- 算法的时间复杂度是一个函数式T(N)，算法中的基本操作的执行次数，为算法的时间复杂度。
- 注：编译器的不同，编译所需要的时间也不同。越新的编译器，编译的时间往往比旧的编译器快
- 当一个算法函数式为`T(N) = N`,和另一个算法函数式为 `T(N) = N^2`比较，必然是第一个快
# 1、大O的渐进表示法
大的渐进表示法的规则：
>1. 时间复杂度函数式T(N)中，只保留最高阶项，去掉那些低阶项(当N无穷大时，低阶项的影响越来越小)
>2. 如果最高阶项是一个一次线性函数，则去除常数系数(当N无穷大时，1的影响很小)
>3. T(N)中如果没有N相关的项目，只有常数项，用常数1取代所有加法常数

我们来判断一段代码的时间复杂度
```C
// 请计算⼀下Func1中++count语句总共执⾏了多少次？

void Func1(int N)
{
	int count = 0;
	for (int i = 0; i < N; ++i)
	{
		for (int j = 0; j < N; ++j)
		{
			++count;
		}
	}
	for (int k = 0; k < 2 * N; ++k)
	{
		++count;
	}
	int M = 10;
	while (M--)
	{
		++count;
	}
}
```
Func1 执⾏的基本操作次数：`T (N) = N2 + 2 ∗ N + 10`
通过对N取值分析，对结果影响最大的⼀项是`N2`
通过以上方法，可以大致评估`Func1`的时间复杂度为：`O(N2 )`

## 2、函数clock计算运算时间
我们想计算代码运算的时间，可以运用clock函数进行计算。**运算过程为运算末-运算初**
```C
#include<stdio.h>
#include<time>

int main()
{
	int i = 0;
	int begin = clock();
	int x = 10;
	for(i = 0; i < n; i++)
	{
		x++;
	}
	int end = clock();
	
	//计算运行时间
	printf("%dms", end - begin);
	return 0;
}
```

## 3、常见复杂度对比

| 5201314         | 0(1)     | 常数阶    |
| --------------- | -------- | ------ |
| 3n+4            | O(n)     | 线性阶    |
| 3n^2+4n+5       | 0(n^2)   | 平方阶    |
| 310g(2)n+4      | 0(1ogn)  | 对数阶    |
| 2n+3nlog(2)n+14 | O(nlogn) | nlogn阶 |
| n^3+2n^2+4n+6   | 0(n^3)   | 立方阶    |
| 2^n             | 0(2^n)   | 指数阶    |

### 3.1常数项复杂度
```C
#include<stdio.h>

int main()
{
	int x = 0;
	scnaf("%d", &x);
	printf("%d", x);
	return 0;
}
```
执行的基本操作次数：`T (N) = 3`
根据推导规则第3条得出时间复杂度为：`O(1)`
### 3.2线性时间复杂度
#### 案例1
```C
// 计算Func2的时间复杂度？
void Func2(int N)
{
	int count = 0;
	for (int k = 0; k < 2 * N; ++k)
	{
		++count;
	}
	int M = 10;
	while (M--)
	{
		++count;
	}
	printf("%d\n", count);
}
```
Func2执行的基本操作次数：`T (N) = 2N + 10`
根据推导规则第3条得出Func2的时间复杂度为：`O(N)`
#### 案例2
```C
// 计算Func3的时间复杂度？
void Func3(int N, int M)
{
	int count = 0;
	for (int k = 0; k < M; ++k)
	{
		++count;
	}
	for (int k = 0; k < N; ++
		k)
	{
		++count;
	}
	printf("%d\n", count);
}
```
Func3执行的基本操作次数：`T (N) = M + N`
因此：Func3的时间复杂度为：`O(N)`
### 3.3平方阶复杂度
```C
#include<stdio.h>

int main()
{
	int x = 0;
	int begin = clock();
	int n = 100000;
	for (int i = 0; i < n; i++)
	{
		for (int j = 0; j < n; j++)
		{
			x++;
		}
	}
	int end = clock();
	printf("%d\n", x);
	printf("%dms\n", end - begin);
	return 0;
}
```
执行的基本操作次数：`T (N) = i * j`
因此：时间复杂度为：`O(N^2)`

### 3.4对数复杂度
```C
void func5(int n)
{
	int cnt = 1;
	while (cnt < n)
	{
		cnt *= 2;
	}
}
```
当n=2时，执行次数为1
当n=4时，执行次数为2
当n=16时，执行次数为4
假设执行次数为x ，则`2x=n`
因此执行次数：`x=log n`
因此：func5的时间复杂度取最差情况为：`O(log2 n)`

### 3.5递归函数
#### 单递归
**递归时间复杂度：所有递归调用次数的累加**
```C
// 计算阶乘递归Fac的时间复杂度？

long long Fac(size_t N)
{
	if(0 == N)
	return Fac(N-1)*N;
}
```
调⽤⼀次Fac函数的时间复杂度为 O(1)，而在Fac函数中，存在n次递归调用Fac函数
因此：return 1;
阶乘递归的时间复杂度为：O(n)
![](图片/bit-2025-11-07-09-16-38.png)

我们再来看一下往递归里加个for循环：此时递归的时间复杂度为：O(n^2)
![](图片/bit-2025-11-07-09-21-07.png)

#### 双递归
![](图片/bit-2025-11-07-09-44-15.png)

# 三、空间复杂度
空间复杂度算的是变量个数，是对一个算法在运行过程中临时占用存储空间大小的量度，同样也使用**大O渐进表示法**。(一般在编程中不考虑空间复杂度，而多用时间复杂度。空间复杂度多运用在嵌入式)

注意：函数运行时所需要的栈空间(存储参数、局部变量、一些寄存器信息等)在编译期间已经确定好了，因此空间复杂度主要通过函数在运行时候显式申请的额外空间来确定
我们先来看一下经典的冒泡排序
## 冒泡排序O(1)
```C
// 计算BubbleSort的时间复杂度？

void BubbleSort(int* a, int n)
{
	assert(a);
	for (size_t end = n; end > 0; --end)
	{
		int exchange = 0;
		for (size_t i = 1; i < end; ++i)
		{
			if (a[i-1] > a[i])
			{
			Swap(&a[i-1], &a[i]);
			exchange = 1;
			}
		}
	if (exchange == 0)
	break;
	}
}
```
函数栈帧在编译期间已经确定好了，只需要关注函数在运行时额外申请的空间。
BubbleSort额外申请的空间有exchange等有限个局部变量，使用了常数个额外空间，因此空间复杂度为 O(1)
## 三个反置O(N)
```C
void reverse(int* nums, int left, int right)
{
	while (left < right)
	{
		int tap = nums[left];
		nums[left] = nums[right];
		nums[right] = tap;
		left++;
		right--;
	}
}

int main()
{
	int nums[] = { 1,2,3,4,5,6,7 };
	int numsSize = sizeof(nums) / sizeof(nums[0]);
	int k = 0;
	scanf("%d", &k);
	k %= numsSize;
	reverse(nums, 0, numsSize - k - 1);
	reverse(nums, numsSize - k, numsSize - 1);
	reverse(nums, 0, numsSize - 1);

	for (int i = 0; i < numsSize; i++)
	{
		printf("%d ", nums[i]);
	}
	return 0;
}
```
- 由于创建了个数组，数组的空间复杂度为O(N)
空间复杂度一般只会出现O(1),O(N),O(N^2)，在复杂度中，还是更看重时间复杂度