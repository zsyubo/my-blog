# Deque

和vector类似，但是相比于Vector，支持首尾插入或者删除元素。

```c++
#include <deque>
using namespace std;
int main()
{
    deque<int> intDeque;
    intDeque.push_back(1);
    intDeque.push_back(2);
    intDeque.push_front(3);
    intDeque.push_front(4);
    // 4 3 1 2
    return 0;
}
```

