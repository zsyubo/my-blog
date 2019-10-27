# Buffer

`Buffer`是NIO三大组件之一，直译为`缓冲区`。通过就是用来缓存从字节流中读取到的数据，当数据读取到缓存区时，程序可以直接对已读取到的缓冲区数据进行业务逻辑处理。

# Buffer体系

![](https://img-blog.csdnimg.cn/20190528191358540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ2NzU1Mw==,size_16,color_FFFFFF,t_70)

当然在JDK中还有很多`Buffer`,比如`IntBuffer、LongBuffer`等等，但是使用最多的还是`ByteBuffer`,这里就不赘述了。

# Buffer核心属性以及相关操作

`mark <= position <= limit <= capacity`

```java
    private int mark = -1;  // 标记位置，用于记录
    private int position = 0; // 下一次读写的问题
    private int limit;  // 可供读写的位置，  用于限制position ，position <limit
    private int capacity; // Buffer所能够存放的最大容量,初始化后就不会在变了。
```















































