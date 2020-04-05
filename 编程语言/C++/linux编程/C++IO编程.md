# IO流程

c++中分为3中输入输出流：

- 标准IO流： 内存与标准输入设备之间信息的传递
- 文件IO流：文件与外部文件之间的信息传递。
- 字符串IO流：内存变量与表示字符串流的字符数组之间信息的传递。

在C++，为了将3中输入输出接口统一起来，使用符号">>" 读取数据，使用符号“<<” 写数据。

# IO流类库

ios为基类，它直接派生4个类

- 输入流：istream
- 输出流：ostream
- 文件流：fstreambase
- 字符串流基类：strstreambase

`C++系统的的IO类库的所有类都被包含在iostream、fstream和strstream3个头文件中`。



# 打开文件

ofstream和fstream对象都可以用来打开文件进行写操作，如果只需要打开文件进行读操作，可以使用`ifstream`

```c++
#include <fstream>

void open(const char *filename, ios::openmode mode);
```

mode为打开模式，具体如下

![](https://s1.ax1x.com/2020/03/27/GiUrkt.png)



如果是需要写入，可以这样写：

```c++
ofstream outfile;
outfile.open("hello.txt",ios::out | ios::trunc);
```

如果只需要读取，可以如下：

```c++
fstream afile;
afile.open("hello.txt", ios::out | ios::in);
```



当然，如果不传mode，那么会采用默认行为，如图

![](https://s1.ax1x.com/2020/03/27/Gi6VHO.png)

还可以定义对象时直接打开文件

```c++
ofstream file("hello.t11xt", ios::out | ios::in | ios::binary);
```

另外，我们可以通过调用成员函数`is_open()`来检查一个文件是否已经被顺利的打开了

```c++
bool is_open();
```

# 关闭文件

读写完成后，我们需要关闭流

```c++
void close();
```

# 写入文件

c++提供了`写入运算符 << `从文件写入信息。

# 读取

c++提供了`提取运算符 >> `从文件读取信息。