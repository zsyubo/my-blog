[Java IO流学习总结一：输入输出流](https://blog.csdn.net/zhaoyanjun6/article/details/54292148)

# IO流结构图

<img src="https://s2.ax1x.com/2019/10/23/KtQu60.png" style="zoom:80%;" />



# `什么是流？`

`流`的本质就是数据传输，即两设备间传输称为流。

按照数据流向可以分为：输入输出流，按照处理的数据类型分为：字节字符流。

**何为输入输出流**

<img src="https://s2.ax1x.com/2019/10/23/KtlLPH.png" style="zoom:50%;" />

**字节流、字符流区别**

在网络传输的数据都是都是经过编码的，这时候直接处理字节流就变得不那么方便的，所以产生了字符流，但是底层还是对字节流的处理，只是去对应的码表查询对应的字符。主要区别如下：

1.  读取单位不同:`字节流以字节(1Byte = 8bit,FF)为单位。字符流以字符为单位，根据码表映射字符，一次可读取多个字节。`
2. 处理对象不同： 字节流能处理任意类型的数据，但是字符流只能处理文本类型数据。
3. 字节流： 一次读取8个二进制位。
4. 字符流：一次读取16个二进制位(随码表的不通过而不同)。

在计算机中，数据都是以二进制存储在计算机中，所以字节流可以处理计算机中所有数据，但是字节流也一样能处理字符数据。`如果只是处理纯文本数据，优先考虑字符流(双方码表必须相同)，除此之外都考虑字节流。`



## 输入字节流 InputStream

- `InputStream` 是所有的输入字节流的父类，它是一个抽象类。

- `ByteArrayInputStream、StringBufferInputStream、FileInputStream` 是三种基本的介质流，它们分别从

  `Byte 数组`、`StringBuffer`、和`本地文件`中读取数据。

- `PipedInputStream` 是从与其它线程共用的管道中读取数据，与Piped 相关的知识后续单独介绍。

- `ObjectInputStream` 和所有`FilterInputStream` 的子类都是装饰流（装饰器模式的主角）。

## 输出字节流 OutputStream

- `OutputStream` 是所有的输出字节流的父类，它是一个抽象类。
- `ByteArrayOutputStream`、`FileOutputStream`是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。
- `PipedOutputStream` 是向与其它线程共用的管道中写入数据。
- `ObjectOutputStream` 和所有`FilterOutputStream`的子类都是装饰流。

## 节点流 

直接与数据源相连，读入或读出。

**常见节点流**

- 父　类 ：`InputStream` 、`OutputStream`、` Reader`、 `Writer`
- 文　件 ：`FileInputStream` 、` FileOutputStrean` 、`FileReader` 、`FileWriter` 文件进行处理的节点流
- 数　组 ：`ByteArrayInputStream`、 `ByteArrayOutputStream`、 `CharArrayReader`、`CharArrayWriter` 对数组进行处理的节点流（对应的不再是文件，而是内存中的一个数组）
- 字符串 ：`StringReader`、` StringWriter` 对字符串进行处理的节点流
- 管　道 ：`PipedInputStream` 、`PipedOutputStream` 、`PipedReader`、`PipedWriter` 对管道进行处理的节点流

## 处理流

直接使用节点流，读写不方便，为了更快的读写文件，才有了处理流。处理流和节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流。如`BufferedReader`.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

**常见的处理流**

- 缓冲流：BufferedInputStrean 、BufferedOutputStream、 BufferedReader、 BufferedWriter 增加缓冲功能，避免频繁读写硬盘。
- 转换流：InputStreamReader 、OutputStreamReader实现字节流和字符流之间的转换。
- 数据流： DataInputStream 、DataOutputStream 等-提供将基础数据类型写入到文件中，或者读取出来。



# 装饰器模式在JavaIO流中使用

## 装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

这种设计模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。

`简单说主要是在不改变原有API的情况对方法进行增强(提供额外功能)`。

## 装饰器在Java IO流中的应用。

以`InputStream`为案例：`InputStream`是一个输入流的抽象类。

```java
public abstract class InputStream implements Closeable {
     public abstract int read() throws IOException;
    	.......
      .......
}
```

比如`read()`是一个抽象方法，交由子类去实现。其子类有

- ByteArrayInputStream
- FileInputStream
- ObjectInputStream
- StringBufferInputStream

这些子类都有自己特定的功能，比如`FileInputStream`就是文件输入流，`ObjectInputStream`提供了对象反序列功能。

InputStream这个抽象类有一个子类与上述其它子类非常不同，这个子类就是`FilterInputStream`,`FilterInputStream`类部维护了一个`InputStream`。

```java
public class FilterInputStream extends InputStream {
    /**
     * The input stream to be filtered.
     */
     protected volatile InputStream in;
  
     protected FilterInputStream(InputStream in) {
          this.in = in;
      }
     public int read() throws IOException {
        return in.read();
      }
   .......... 
}
```

内部代码基本上什么也没做，就是调用传入的`InputStream的同名方法。`

`FilterInputStream`的又有其子类，分别是：

- BufferedInputStream
- DataInputStream
- LineNumberInputStream
- PushbackInputStream

虽然`FilterInputStream`没做什么事,主要定义了构造方法，以便实现装饰器模式。但是他的子类就是干实事的了。比如`BufferedInputStream`

```java
public class BufferedInputStream extends FilterInputStream {
        public BufferedInputStream(InputStream in, int size) {
            super(in);
            if (size <= 0) {
                throw new IllegalArgumentException("Buffer size <= 0");
            }
            buf = new byte[size];
        }
        public synchronized int read() throws IOException {
            if (pos >= count) {
                fill();
                if (pos >= count)
                    return -1;
            }
            return getBufIfOpen()[pos++] & 0xff;
        }
    .............
    .............
}
```

从代码中可以看到，`BufferedInputStream`就是一个装饰者，原来的`InputStream`并有缓存区功能，但是`BufferedInputStream`通过装饰者模式为`InputStream`添加上了缓存区功能，而且并没有改变原有API。

> Ps：`fill`最终会调用`InputStream`的`public int read(byte b[], int off, int len)`。
>
> 参考来源：[Java IO : 流，以及装饰器模式在其上的运用](https://segmentfault.com/a/1190000004255439)

