# list

`list`是一个双向链表，链表的优点是插入、删除的速度是o(1)，但是查找是o(n)。

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

    vector<int> vecInt(10,99);
    list<int> listCopyVector(vecInt.cbegin(),vecInt.cend());
}
```

