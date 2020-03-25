# 支持c++ 11、17、20

g++默认是无法编译11以后的新特性，这时候我们需要在编译命令加如下参数。

```shell
g++ -std=c++17  xxx.cpp
```

当然，我们可以加入命令别名：

```shell
> sudo vim ~/.bashrc 
alias g++11='g++ -std=c++11'
alias g++14='g++ -std=c++14'
alias g++17='g++ -std=c++17'
alias g++20='g++ -std=c++2a'
```

