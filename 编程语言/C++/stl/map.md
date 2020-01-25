# map

map和multimap是键-值容器，支持根据键进行查找。

<img src="https://s2.ax1x.com/2020/01/06/lyUx58.png" style="zoom:40%;" />

使用map需要包含头文件

```c++
#include <map>
```

map 和 multimap 之间的区别在于，后者能够存储重复的键，而前者只能存储唯一的键。

# 基本操作

## 初始化

```c++
  // 默认方式
    map<string ,string> mapObj1;
    // 指定key的排序规则
    map<string,string ,std::less<string>> map2;

    map<string ,string> mm3(mapObj1);

    map<string,string> mm4(map2.cbegin(), map2.cend());

    map<string,string,std::greater<string>> mm5(map2.cbegin(), map2.cend());
```

## 插入

```c++
  mapObj1.insert(make_pair("name","bob"));
    mapObj1.insert(pair<string,string>("sex","鸟人"));
```

## 查找

```c++
    map<string,string>::const_iterator pairFound=mapObj1.find("sex");
    cout << pairFound->first <<endl;
    map<string,string>::const_iterator pairFound1=mapObj1.find("se2x");
    cout << pairFound1->first <<endl;// 返回空
```

如果是multimap，那么可以通过`pairFound1.operator++`来访问下一个相同key的。

## 删除

```c++
 mapObj1.erase("name");
// 迭代器
mapObj1.erase(xx.begin(),xx.end());
```

## 自定义排序

```c++
template<typename keyType>
    struct Predicate
    {
       bool operator()(const keyType& key1, const keyType& key2)
       {
          // your sort priority logic here
         return key1>key2;
       }
};
```

# unordered_map 和 unordered_multimap

基于散列