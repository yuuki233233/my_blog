```C
int main()
{
	int num;
	
	const int *p1 = &num;//const修饰*p1 即num
	//错误
	//(*p1)++
	p1++
	
	int *const p2 = &num;//const修饰p2
	//错误
	//p2++
	(*p2)++;
	
	const int *const p3 = &num;//都不能修改	
}
```
在做形参的时候，希望某个参数不被修改的话，可以在前面加const