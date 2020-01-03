# list

`list`是一个`双向链表`，链表的优点是插入、删除的速度是o(1)，但是查找是o(n)。

![](https://note.youdao.com/yws/api/personal/file/WEB9b79f0cdc1316fb5cc5c88e29dbd884e?method=download&shareKey=74df10e0f198a087a97a5780a3e7a200)

链表是一系列节点，其中每个节点除包含对象或值外还指向下一个节点，即每个节点都链接到下
一个节点和前一个节点。

使用`list`需要引入对应头文件

```c++
#include <list>
```

# 实例化

`list`的实例化方式和`vector`差不多。

```c++
#include <list>
#include <vector>
using namespace std;
int main()
{
    list<int> linkInts;

    list<int> listWith10Int(10);

    list<int> listWith4IntEach99(10,99);

    list<int> listCopyAnother(listWith4IntEach99);
		// 还可以通过迭代器，从其他容器复制数据
    vector<int> vecInt(10,99);
    list<int> listCopyVector(vecInt.cbegin(),vecInt.cend());
}
```

# API

## 插入

**开头、末尾**

和deque一样的函数

```c++
    list<string> names;
    names.push_front("10");
    names.push_front("20");
    names.push_front("30");
    names.push_back("40");
    names.push_back("50");
    // 30 20 10 40 50	
```

**任意位置插入**

和vector差不多，不过需要注意的是不能直接使用`begin()+偏移量的做法`

```c++
    // 报错
     names.insert(names.cbegin()+1, "gg"); 
    // 但是可以这样写。。
    names.insert(++names.cbegin(), "gg");
```

## 删除

**迭代器删除**

```c++
names.erase(names.begin());
iteratorContainer(names);
// 通过迭代器，范围删除 []区间。   
names.erase(++names.begin(),names.end());    
iteratorContainer(names);

```

**清空**

```c++
names.clear();
```

## 反转list

反转list的排列顺序。

```c++
names.reverse();
```

**排序**

```C++
// 无任何操作。并不会更改元素排列顺序
names.sort();
```

我们可以传一个函数，从而指定他的排序规则

```c++
// 实现反转list
bool SortPredicate_Descending (const int& lhs, const int& rhs)
{
   return (lhs > rhs);
}

int main(){
    list<int> names;
    names.push_back(10);
    names.push_back(20);
    names.push_back(30);
    names.push_back(40);
    names.push_back(50);
    names.push_back(60);

    names.sort(SortPredicate_Descending);
    iteratorContainer(names); // 60 50 40 30 20 10
    return 0;
}
```

