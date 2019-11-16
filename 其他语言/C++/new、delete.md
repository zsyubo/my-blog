# new的不同写法

```c++
Person p11;// 默认不进行初始化，如果直接读取数据会编译失败，需要先手动初始化。 如果在全局区，会对对象字段进行初始化0值。	

	Person *p1 = new Person;
	Person *p2 = new Person();
```

看上区别只是在括号上，但是实际区别挺大的。

```assembly
	Person *p1 = new Person;
000E2BD8  push        4Ch  
000E2BDA  call        operator new (0E13BBh)  
000E2BDF  add         esp,4  
000E2BE2  mov         dword ptr [ebp-0ECh],eax  
000E2BE8  cmp         dword ptr [ebp-0ECh],0  
000E2BEF  je          main+54h (0E2C04h)  
000E2BF1  mov         ecx,dword ptr [ebp-0ECh]  
000E2BF7  call        Person::Person (0E1163h)  
000E2BFC  mov         dword ptr [ebp-100h],eax  
000E2C02  jmp         main+5Eh (0E2C0Eh)  
000E2C04  mov         dword ptr [ebp-100h],0  
000E2C0E  mov         eax,dword ptr [ebp-100h]  
000E2C14  mov         dword ptr [p1],eax  

Person *p2 = new Person();
000E2C17  push        4Ch  
000E2C19  call        operator new (0E13BBh)  
000E2C1E  add         esp,4  
000E2C21  mov         dword ptr [ebp-0F8h],eax  
000E2C27  cmp         dword ptr [ebp-0F8h],0  
000E2C2E  je          main+0A6h (0E2C56h)  
000E2C30  push        4Ch  
000E2C32  push        0  
000E2C34  mov         eax,dword ptr [ebp-0F8h]  
000E2C3A  push        eax  
// 这而需要注意额是，如果对象字段太少会做编译优化，直接一个一个区初始化。
000E2C3B  call        _memset (0E113Bh)  
000E2C40  add         esp,0Ch  
000E2C43  mov         ecx,dword ptr [ebp-0F8h]  
// 调用默认构造方法，注意，如果对象字段只是基础类型(int、long等)，那么在调用memset后不会在调用默认构造函数了(前提是代码中没定义构造函数)。
000E2C49  call        Person::Person (0E1163h)  
000E2C4E  mov         dword ptr [ebp-100h],eax  
000E2C54  jmp         main+0B0h (0E2C60h)  
000E2C56  mov         dword ptr [ebp-100h],0  
000E2C60  mov         ecx,dword ptr [ebp-100h]  
000E2C66  mov         dword ptr [p2],ecx  
```

看以看出，有括号和没括号的汇编代码还是区别挺大的。最核心的区别在`call        _memset (0E113Bh)  `，`memset`是吧一段内存空间初始化为0值，也就是有括号会把内存空间初始化为0值。

>  memset函数是将较大的数据结构（比如对象、数组等）内存清零

## 关于编译器生成默认无参构造函数

调用默认构造方法，注意，如果对象字段只是基础类型(int、long等)，那么在调用memset后不会在调用默认构造函数了(前提是代码中没定义构造函数)。

或者：

```c++
class Person {
public:
	int age=0;
	}
```

当手动设置初始值时，编译器也会默认生成一个构造函数，`age`在构造函数中进行赋值。



# delete

和`free`，