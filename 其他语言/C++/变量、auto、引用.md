# # 变量
在c++中，局部变量及初始化可以做到随时定义、初始化。
## ## 新定义方式
```c++
int i{ 5 };
int i = { 5 };
// 数组可以这样定义
int a[]{ 11,12,35 };
```
有意思的地方
```c++
// 程序正常执行，但是 `.5f`会被系统截断。
int  ac = 3.5f
// 直接编译失败
int ac {3.5f}
```

# # auto 变量的自动类型推断
可以在变量声明的时候根据变量的初始值的类型自动为此变量选择匹配的类型。当然啦，自动推断是发生在`编译期间`,所以既然要推断，`声明时就要赋予初始值`，可能编译时慢一点，但是好处是`运行时不会造成程序性能降低`。
```c++
 auto bb = true;
 auto cc { 1 };
```

# # 引用
可以理解为：为变量启另一个名字(`别名`)，一般用&符号表示。起完别名后 ，这别名和变量可以看做是同一变量。`定义引用的时候必须初始化`,同时要与别引用的类型相同。底层其实并不是占的同一块内存。`引用不能修改指向`。
```c++
 int value = 110;
// 注意这不是取地址，是引用。
 int &ref = value;
// 和普通变量一样使用
 cout << ref << endl;

// 指针
int *p= &value;
// 指针的引用
int *&ref = p;
*p=30;

//数组的引用
int array[] = {1,2,3};
// 必须写三，也就是数组大小
int (&ref)[3] = array;
ref[0]= 99;
```
引用是指针的弱化版(汇编一样，只是编译器做了限制)，更安全。  一个引用占用一个指针的大小 。指针的汇编代码和引用一样的。

一般用于参数传递过程中，直接修改变量值。

```c++
int main()
{
int a = 15; 
 int b = 61;
 // 非方法传参则会影响原值。
 func(a, b);
 cout << "a:" << a << "b:" << b << endl; // 4,5
 int &a1 = a;
 int &b1 = b;
 a1 = 99;
 b1 = 88;
 cout << "a:" << a << "b:" << b << endl; // 99,88
}

void func(int &a, int &b) {
// 在函数值中修改引用值，对原值有影响。
 a = 4;
 b = 5;
}
```
当然也可以用指针，不过要麻烦一些。

> 引用是一个指向其它对象的常量指针，它保存着所指对象的存储地址。并且使用的时候会自动解引用，而不需要像使用指针一样显式提领。

##  常引用

```c++
const int &ref = 30;
```

`const与引用结合使用过时，其实和指针是差不多的。

常引用用处挺多的，比如传参

```c++
int sum( const int &v1, const int &v2){
	return v1 + v2;
}
// 直接传值时，函数参数必须是常引用
sum(10,20);
```

`const`修饰的参数名可以接受`const`和非`const`实参。而非`const`引用只能接收非`const`实参。可以和`非const`引用构成重载:

```c++
int sum( const int &v1, const int &v2){
	return v1 + v2;
}
int sum(int &v1, int &v2){
	return v1 + v2;
}
```

测试代码

```c++
	{
		int a = 10;
		int b = 20;
		std::cout  << sum(a, b) << std::endl;
	}
	{
		const int a = 10;
		const int b = 20;
		std::cout << sum(a, b) << std::endl;
	}
```

结果

```
sum( int &v1,  int &v2)
30

sum(const int &v1, const int &v2)
30
```

## 常引用引发的一些奇怪问题

```c++
	int rage = 1;
	const long &rang = rage;
	rage = 2;
	std::cout << "rage："<< rage << std::endl;  // 2
	std::cout << "rang：" << rang << std::endl;  // 1
```

结果与我们想的30,30不一样，通过看汇编代码知晓原因

```assembly
// int rage = 1
00136056  mov         dword ptr [ebp+FFFFFF70h],1  
//  取 rage 的值 到eax
00136060  mov         eax,dword ptr [ebp+FFFFFF70h]  
// 把 eax的值 写入到新地址
00136066  mov         dword ptr [ebp+FFFFFF58h],eax  
// 把 刚才写入的新地址传给exc
0013606C  lea         ecx,[ebp+FFFFFF58h]  
// 把exc地址的写入到 rang引用
00136072  mov         dword ptr [ebp+FFFFFF64h],ecx  
// 	rage = 2;
00136078  mov         dword ptr [ebp+FFFFFF70h],2 
```

从这里看出:`	const long &rang = rage;`不是直接吧 `rage`的地址给`&rang`，而且先把数据拷贝了一份到一个新地址，然后吧新地址传个`&rang`，所以修改`rage`对`&rang`无效。