# `vector

`vector`是一个模板类，提供了额动态数组的通用功能。使用需要引入头文件`#include <vector>`

# vector初始化

## 实例化

```c++
    vector<int> vet;
    vector<int> vv={1,23,4};
    cout << vv.size() << endl;//3 
```

当我们需要拷贝，除了拷贝构造函数也可以这样做

```c++
   vector<int>v2={vv.begin(),vv.end()};
    cout << v2.size()<<endl;
```

**全部初始化方式**

```c++
#include <iostream>
#include <vector>
#include <string>
using namespace std;

int main()
{
 		// 初始化一个空的vector
    vector<int> integers;
 		// C++11支持，在初始化时指定容易有3个元素
    vector<int> initVector{202,2018,-1};
   // 指定容器的初始大小为10,每个元素初始化为0
    vector<int> tenElements(10);
		// 创建一个vector，元素个数为10,且值均为90
    vector<int> tenElemInit(10,90);
 		// 复制构造函数
    vector<int> copyVector(tenElemInit);
		// 复制[begin,end)区间内另一个数组的元素到vector中
    vector<int> partialCopy(tenElements.cbegin(),
          tenElemInit.cbegin()+5);
}
```

# API

## push_back 从尾部插入数据

使用`push_back`我们可以插入数据到vector尾部。

```c++
    vector<string> name;
    name.push_back("bob");
```

## insert 插入元素到指定位置

```c++
vector<string> names={"bob","framk","jams","ben"};
// insert(位置偏移量，插入值)
names.insert(names.begin()+1,"kuke");
// insert(位置偏移量，插入元素个数，插入值);   
names.insert(names.begin()+1,2,"finn");// bob finn finn kuke  framk.....
vector_iteartor::iteratorContainer(names);
cout << "-----------"<< endl;
vector<string> nums={"1","2","3"};
// 把一个容器插入到另一个容器的指定位置。
names.insert(names.end(),nums.begin(),nums.end()); // .... framk jams ben 1 2 3
vector_iteartor::iteratorContainer(names);
```

## 使用数组语法访问

我们可以使用访问数组的方式来访问vector

```c++
void ff(vector<string> &nn)
{
    for(size_t index =0 ; index < nn.size(); index++ )
    {
         if(index==2){
            nn[index]="三版本";
        }
        cout << nn[index]<<endl;
    }
}
```

当然，我们在访问时，不能发生数组越界行为，否则报错。

## pop_back 删除vector末尾元素

```c++
    vector<string> names ={"finn","jack","ben","bob"};
    cout << "name.size()=" << names.size() << endl; // 4
    names.pop_back();
    cout << "name.size()=" << names.size() << endl; // 3
```

## erase 删除指定位置元素

```c++
    vector<string> names ={"finn","jack","ben","bob","sam","lam"};
    cout << "name.size()=" << names.size() << endl;
   // 删除头部元素
    names.erase(names.begin());
    cout << "name.size()=" << names.size() << endl;
    vector_iteartor::iteratorContainer<string>(names);
    cout << "-----------------"<< endl;
		// 从头部开始，删除names.begin()、names.begin()+1 两个元素  [names.begin(),names.begin()+2)
    names.erase(names.begin(),names.begin()+2);
    vector_iteartor::iteratorContainer<string>(names);
```

# 容量与大小

如果需要查询`容器存储的元素数量`,可以使用`size()`函数

```c++
cout << "Size: " << integers.size ();
```

要查询` vector`的容量，可调用`capacity( ):`

```c++
cout << "Capacity: " << integers.capacity () << endl;	
```

`capacity`是实际容器的底层数组大小，当元素个数大于当前`capacity`时，容器就需要扩容了，扩容需要多余的开销，当我们能确定容器要放置多少数据时，我们可以使用`reserve`函数指定`capacity`，这样就不用频繁扩容。

```c++
    vector<string> names ={"finn","jack","ben","bob","sam","lam"};
    names.pop_back();
    names.pop_back();
    cout << "cap:" << names.capacity() << endl; // 4
    cout << "size:" << names.size() << endl; // 6
    // 指定 capacity
    names.reserve(10);
    cout << "cap:" << names.capacity() << endl; // 10
    cout << "size:" << names.size() << endl; // 6
```

**扩容原理**

todo