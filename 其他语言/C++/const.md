# 1. 常量

表示此变量不可以修改。很多时候常用来替代宏。全局变量和类数据成员都可以用`const`修饰

且必须定义时进行初始化，或者构造函数中初始化。

```c++
 const int var1 = 7;
 cout << "var1:" << var1 << endl;
// 报错。
 int &var11 = var1;
// 也有方法搞，直接强转。
 int &var11 = (int &)var1;
 var11 = 666666;
 cout << "var1:" << var1 << endl; // 7
 cout << "var11:" << var11 << endl; // 666666
// 但是在debug模式下，var1、var11都为666666，编译器做了一些优化的。
```
其实就是类似于java中的`final`,但是还是有不同：

```c++
	const A* a = new A();
	a->age = 10; // 编译失败，const修饰的变量不能修改其成员属性值。
```

## const 与类

当一个实例使用const表示时，那么此实例无法调用非const方法，也就是`无法调用普通实例方法`。

## const成员方法

```c++

class Student {
public:
	void say() const {
	}
};
```

在`const`修饰的成员函数不能修改非`static`修饰的成员变量。

`cosnt`修改的成员函数与非`const`修饰的成员函数之间构成`重载`。

# 2. const方法参数

一般我们在写方法接受对象时，会这样写。多用于不用修改这个值。

```c++
void say(const Point &p1) const {
	.....
}
```

为什么这样写了，因为`const`能接受的范围更大，可以接受`cosnt`和`非const`参数。

> 将对象作为参数传递时，默认选择是`const`引用。只有在明确需要修改对象时，才忽略`const`。

# 3. constexpr 关键字，c++11引入

也一样是常量概念。在编译的时候求值，好处是提升性能。`constexpr` 等式左右必须是常量

 constexpr int var2 = 1;
 constexpr auto var22 = 1;

当然也可以这样定义
```c++
// 必须加 constexpr 
constexpr int funcC(int a) {
 return a--;
}

int main()
{
 constexpr auto var2233 = funcC(3);
}
```

# 4. const 指针

我们知道指针分为2部分，一部分是指向的数据，一部分是指针本身(地址)。所以在使用const修饰时就会变得微妙。到底是不能让指针指向的地址不能改还是指向的数据不能改？

**const int* ip **

无法改变ip指向的值，语义上等价`int const* ip;`

**int* const ip**

这是将ip本身标记为`const`，不能修改所指向的地址。这是因为ip不能修改，所以必须在声明时进行初始化。

**const int* const ip**

进步修改指针指向的地址，也不能修改所指向的值。



# 5. const 引用

因为const是默认无法改变引用所指向的对象的。所以加`const`的情况和指针有点区别。引用的const只能加在最前面。

同时加了`const`就意味着无法修改对象所指向的值。

```
int z =0;
const int& zRef = z;
zRef = 4; // 编译失败。
```



****

# 6. const char *, char const * , char * const 区别

<p style="color:red"> const是修饰其右边的内容 </p>
## ## const char *  
```c++
const char *   p
```
p指向的东西不能通过p来修改（p所指向的目标，那么目标中的内容不能通过p来改变。）
也就是可以改变指向。

`也就是p不是常量，*p是常量`

## ##  char const *
const char *等价于char const * 

## ## char * const 
```c++
char * const    p
```
不可以修改 p指向的内存地址，但是可以修改p指向地址的内容。`也就是 p是常量，*p不是常量`。
## ## const  char * const 
```c++
const char * const  p
```
 p的指向和p指向的内容也不能改变。

也就是`p和*p都是常量。`

# ~~通过引用来让指针指向值无法修改~~

```c++
int age =10;
const int &ref = age;
const int *p = &age;
```

