# string

C提供了字符串，但是没有提供字符串类型，也即是字符数组和`char *`。但是并不好用，同时c在操作字符串时，比较不方便，特别是一些不安全的标准函数，比如`strcpy`，同时如果比较两个字符串也比较麻烦。

> 不安全的原因：这些函数不检查边界，极易造成栈溢出。

c++为我们提供了`std::string`类。

```c++
#include <string>

using namespace std;

int main(int argc, const char * argv[]) {
	// std::string("a");
		string("a");
  	string a("c")
}
```

当然也可以常量字符串初始化string

```c++
    const char * cc="Hello world";
    string* a = new string(cc);
```

我们也可以通过迭代器来遍历string

```c++
// for (string::iterator ai=a->begin(); ai!=a->end(); ai++) 
for (auto ai=a->begin(); ai!=a->end(); ai++) {
        cout<<"1:"<<*ai<<endl;
    }
```

**同时我们很方便进行字符串相加(也可以使用append函数)**

```c++
    string a("a");
    string b("c");
    string c = a + b; // ac
    c+=b; //acc
```

**比较两字符串是否相等**

```
if( a==b ){  // false
 .... 
}
```

**数值转换**

```c++
long  double d = 3.14L;
string s = to_string(d);
// 转回去
long i  stol(s);
```

每种基础类型都提供了，这里就不贴其他的了。

**截取字符串**

```c++
    a->erase(3,6);
// 字节修改的原字符串
    cout<< *a<< endl;
```

当然重载的api还有很多

**查找字符串**

```c++
    size_t index = a->find(b);
    cout << index << endl;
    // 如果未匹配到字符串，则会返回一个特殊数值，也就是string::npos
    cout << string::npos << endl;
```

## 非标准字符串

除了C++标准为，还有一些其他的字符串类型，比如MFC的`CString`，这里就不细说了。







































