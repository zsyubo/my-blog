# 控制台输入输出流

c++中`<<和>>`来实现控制台的输出、输入。c++提供如下预定义流

C++中通过了流`stream`的机制提供了更方便简洁输入输出方法。

| cin  | 输入流，从控制台读取数据         |
| ---- | -------------------------------- |
| cout | 缓冲的输出流，想控制台写入数据   |
| cerr | 非缓冲的输入流，一般用于打印错误 |
| clog | 缓冲的输入流，一般用来打印错误。 |

都定义在`<iostream>`中。

## **cout**

cout是写入控制台的内建流，控制台也可以叫标准输出。要使用输入cout，最简单的方式时`<< `。 使用`<<`可以输出C++的基本类型，如`int、指针、字符串`等。甚至可以多个`<<`串联，从而输出多个数据段。

## **cin**

cin是从控制台读取数据，对应的运算符是`>>`

```java
string cinInput; // 当然也可以读取到 char数组。
cin >> cinInput;// 阻塞, 只会读取回车。
```

需要注意的是，cin是从头读取到空格后就不会继续读取了，也就是输入了`as ds c`回车之后，`cinInput`的值为`as`，空格后的数据全部丢弃。

**getline()**

获取一行数据，它会读取一行数据，直到行尾，当然`行尾字符不会出现在字符串中`。行尾字符视平台不同而不同。

```c++
  string  name;
  getline(cin , name);
  cout<< "Hi,"<< name << endl;
```

**错误处理**

todo



# 文件流

`cin`和`cout`只是在控制台输入输出时使用，在对文件处理则无法使用。C++提供了`std::fstream`的方式来访问文件，提供了读写文件的功能。`fstream`继承了`std::ofstream`写入文件的功能和`std::ifstream`读取文件的功能。

```c++
#include <fstream>
using namespace std;
int main() {
    std::fstream myFIle;
    myFIle.open("ReadText.txt", std::ios_base::in|std::ios_base::out|std::ios_base::trunc);
  // 出了open 还可以使用构造函数方式
  // fstream myFile("xxx.txt", std::ios_base::in|std::ios_base::out|std::ios_base::trunc)
    if( myFIle.is_open() ){
        std::cout << "file is open" << std::endl;
        myFIle.close();
    }
    return 0;
}
```

`open`接受两个参数：第一个是要打开的文件的路径和名称(无路径则默认当前文件夹)，第二个是打开模式：

| 打开模式              | 说明                                   |
| --------------------- | -------------------------------------- |
| std::ios_base::app    | 新写入数据是从末尾还是追加，非覆盖。   |
| std::ios_base::ate    | 打开文件，打开之后立即移动到文件末尾   |
| std::ios_base::binary | 以二进制模式执行输入和输出操作         |
| std::ios_base::in     | 只读                                   |
| std::ios_base::out    | 打开文件，从头开始写入，覆盖已有的数据 |
| std::ios_base::trunc  | 导致现有文件被覆盖，默认               |

**读取文本文件**

```c++
    std::fstream myFIle("ReadText.txt", std::ios_base::in);
    if( myFIle.is_open() ){
        std::cout << "file is open" << std::endl;
        string name ;
        while(myFIle.good()){
            getline( myFIle, name);
            std::cout << "read: "<< name << endl;
        }
        myFIle.close();
    }
```

**写入**

```c++
    std::fstream myFIle("ReadText.txt", std::ios_base::in | std::ios_base::out );
    if( myFIle.is_open() ){
        std::cout << "file is open" << std::endl;
        string name ="dasdadasdsadadasd";
        myFIle << name << endl;
        myFIle.close();
    }
```



# 异常

IO流在使用时肯定会遇到一些问题，比如读取文件文件怎么知道是否读取完了、文件是否存在、是否写入成功等。C++为我们提供了一些状态。

| 函数     | 意义                                                         |
| -------- | ------------------------------------------------------------ |
| 流.bad() | true表示系统级的故障，如无法恢复的读写错误。                 |
| fail()   | true表示遇到错误，但是程序可以去解决，                       |
| eof()    | 遇到文件结束符时设置为true                                   |
| good()   | 当上面三个没有一个为true时，good就为true                     |
| clear()  | 会将 操作设置为有效状态，比如上面的遇到fail时，解决后可调用此函数恢复。 |

































