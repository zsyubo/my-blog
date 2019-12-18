# vector

`vector`是一个模板类，提供了额动态数组的通用功能。使用需要引入头文件`#include <vector>`

# vector使用

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

