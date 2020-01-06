# set和multiset

set是[C++标准库](https://baike.baidu.com/item/C%2B%2B标准库/8795193)中的一种关联容器。所谓关联容器就是通过键（key）来读取和修改元素。与map关联容器不同，它只是单纯键的集合，同时是经过`排序的`,这样有一个`缺点，就是不能通过迭代器来修改元素值`。

和数组`功能相似`，但是底层使用数据结构不同，`set`底层使用红黑树来存储。

`set`不允许重复元素，但是`multiset`允许。

使用`set`和`multiset`只需要引入一个头文件就行

```c++
#include <set>			
```

## 初始化

最基本的初始化方式

```c++
		set<int> setInts;
    multiset<int> meInts;
```

在构造的时候，可以传入一个拷贝结构体，里面定义了具体的排序规则

```c++
    set<int ,SortDescending<int>> setInts2;
    multiset<int ,SortDescending<int> >msetInts2;

// 自带的升序
 multiset<int ,less<int> >msetInts2;
// 自带的降序
 multiset<int ,greater<int> >msetInts2;
```

我们也可以使用拷贝构造函数，但是注意的是，我们不知直接使用比如`set`来构造`multiset`,要使用迭代器来进行拷贝构造。

```c++
    set<int> setInts2(setInts);
    multiset<int > mestInts3(setInts.begin(), setInts.cend());
```

# 使用

## 插入

```c++
   set<int > ints;
   // 顺序插入,虽然我不是按顺序插入的，但是打印还是按照顺序存储的。
    ints.insert(2);
    ints.insert(1);
    ints.insert(4);
    ints.insert(3); 
     Disprint(newOneints); // 1 2 3 4
    cout << endl;
		// 迭代器插入
    set<int> newOne;
    newOne.insert(ints.cbegin(),ints.cend());
    Disprint(newOne);
```

## 查找

反对对应键的迭代器。

```c++
 auto elemnt = ints.find(6);
```

## 删除

set可以根据 元素值删除和迭代器删除，也只是迭代器范围删除

```c++
   	// 删除对应键, 如果是multiset，则删除对应的全部。
		ints.erase(2);
		// 删除迭代器对应元素
    ints.erase(ints.begin());
		// 删除范围内元素
    ints.erase(ints.begin(),ints.end());
```

## count

返回键数量

```c++
    cout << ints.count(2)<<endl;
```

# unordered_set和unordered_multiset

api差不多，底层实现原理不一样，这底层是散列，也就是java中的hashset。