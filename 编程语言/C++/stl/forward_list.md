# Forward_list

是一个`单向链表`,即只允许往一个方向变量。

![](https://s2.ax1x.com/2020/01/03/ldZbeU.png)

使用需要包含头文件

```c++
#include <forward_list>
```

`forward_list`和`list`很像，相当于简化版的`list`。

`forward_list` 的优点在于，它是一种单向链表，占用的内存比 list 稍少，因为只需指向下一个元素， 而无需指向前一个元素。

# API

## 插入

`forward_list`只支持一个方向移动迭代，所以插入元素时只能使用`push_front`。如果需要在任意位置插入，同样可以使用`insert()`。

## 迭代

`forward_list`只能单向迭代，即可以使用`++`, 不能使用`--`。