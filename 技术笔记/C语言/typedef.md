## typedef类型
---
为基本类型创建更有意义的名称(提高可读性)
```C
#include<stdio.h>

typedef int UserId;
typedef char UserName;

int main()
{
	//UserId = int
    UserId my_id = 1001;
    
    //UserName = char
    UserName my_name = "zhangsan";
}

```

简化代码
```C
typedef struct sdu
{

}
```