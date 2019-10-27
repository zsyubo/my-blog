# Buffer

`Buffer`是NIO三大组件之一，直译为`缓冲区`。通过就是用来缓存从字节流中读取到的数据，当数据读取到缓存区时，程序可以直接对已读取到的缓冲区数据进行业务逻辑处理。

`Buffer`底层就是一个类似数组的连续内存空间，但是封装了数据，提供了一些方便的API，更容易管理Buffer 
就像C语言申请内存一样。 

# Buffer体系

![](https://img-blog.csdnimg.cn/20190528191358540.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjQ2NzU1Mw==,size_16,color_FFFFFF,t_70)

当然在JDK中还有很多`Buffer`,比如`IntBuffer、LongBuffer`等等，但是使用最多的还是`ByteBuffer`,这里就不赘述了。

# Buffer核心属性

```java
    private int mark = -1;  // 标记位置，用于记录，可以通过rest()方法回到这里
    private int position = 0; // 下一次读写的问题
    private int limit;  // 可供读写的位置，  用于限制position ，position <limit
    private int capacity; // Buffer所能够存放的最大容量,初始化后就不会在变了。
```

他们之间的关系`mark <= position <= limit <= capacity`

**转换关系**

![](https://s2.ax1x.com/2019/10/27/Kyi9nx.png)

`ByteBuffer`在刚创建是，默认是写入模式，同时`position=0`，在写入模式下，`limit==capacity` 
通过`flip()`函数，可以重写入模式转入到读取模式，`position`变为0，`limit`为写入的最后位置。如果调用`compact()`，则会吧已读取的数据清空(position位置之前)，然后未读取到的数据移动到首位。同时`position`位置变为未读取数据末尾。 

<img src="https://s2.ax1x.com/2019/10/27/KyiJ3j.png" style="zoom:50%;" />

调用`compact()` ,继续写入。

<img src="https://s2.ax1x.com/2019/10/27/Kyi55D.png" style="zoom:50%;" />

# 基础API

1. 申请`Buffer`

```java
 ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4); // 堆外内存
 ByteBuffer byteBuffer = ByteBuffer.allocate( 4 ); // 堆内存
```

2. 将数据写入缓存区

```
byteBuffer.put((byte) 1); // 手动返回一个Byte
```

3. 调用`buffer.flip()`,转换为读取模式

4. 从缓冲区读取数据

```
byteBuffer.hasRemaining(); // 是否还有未读取的数据(未读取到末尾)
byte a = byteBuffer.get();
```

5. 调用`buffer.clear()`或`buffer.compact()`清除已读取的缓冲区。

## 基础API源码

**flip**

```java
    public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```

很直观了，直接把`position`的值给`limit`， 同时`position`初始化为0。



# 拓展

## 堆外内存

在`ByteBuffer`中, 在初始化调用`allocateDirect();`会得到`DirectByteBuffer`。底层是使用的`unsafe`来进行内存分配的。

```java
    long address;// 存放对应开启的空间地址。
    // Primary constructor
    //
    DirectByteBuffer(int cap) {                   // package-private
        super(-1, 0, cap, cap);
        boolean pa = VM.isDirectMemoryPageAligned();
        int ps = Bits.pageSize();
        long size = Math.max(1L, (long)cap + (pa ? ps : 0));
        Bits.reserveMemory(size, cap);

        long base = 0;
        try {
            base = unsafe.allocateMemory(size);
        } catch (OutOfMemoryError x) {
            Bits.unreserveMemory(size, cap);
            throw x;
        }
        unsafe.setMemory(base, size, (byte) 0);
        if (pa && (base % ps != 0)) {
            // Round up to page boundary
            address = base + ps - (base & (ps - 1));
        } else {
            address = base;
        }
        cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
        att = null;
    }
```

## Buffer数据拷贝案例

```java
public static void main(String[] args) throws IOException {
        FileInputStream inputStream = new FileInputStream("buffer.txt");
        FileOutputStream outputStream = new FileOutputStream("buffer2.txt");

        FileChannel input = inputStream.getChannel();
        FileChannel output= outputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate( 512 );
        while ( true){
            byteBuffer.clear();
            //read()方法返回的int代表着有多少数据被读入了Buffer。如果返回-1，代表着已经读到文件结尾。
           int read =  input.read( byteBuffer );
            if ( -1 == read ){
                break;
            }
            byteBuffer.flip();
            output.write( byteBuffer );
            outputStream.flush();
        }
    }
```

这里说一下` byteBuffer.clear();`，如果注释掉这段代码会发生什么？

`read`方法会一直返回`0`,从而造成一直循环写。

**为什么这样了？**

很简单，当程序进来第一次读取写入，正常，在进行第二次读取的时候：`limit`和`position`是一样，我们知道：`limit`是永远大于等于`position`，同时读取数据时移动的是`position`，但是这是`limit`和`position`是一样，所以`position`无法再移动了，所以不会读取到任何数据，最终`read`返回0,读取到了`0`个字节的数据。

## 为什么要堆外内存？

1. `Direct buffer`是java申请的用户空间，相当于`C`中的`malloc`申请的内存。
2. 使用`Direct buffer`减少了一次拷贝： 在`非Direct buffer`时，JDK也会前创建一个`Direct buffer`，再在Java对中创建一个`Buffer`，之后再去执行真正的读写操作。为什么了？因为操作系统在读写时，内存地址不能失效，但是处于Java堆管理的`buffer`可能会由于GC的问题，导致对象移动(比如带有整理算法的垃圾收集器)，这时候对象地址就变量，操作系统就无法进行读写了。所以需要一个GC管不了的地方，所以好处就是`减少了一次拷贝`，至于减少内存，倒不一定，因为堆外的`buffer`在读写完成后移动到java堆中。
3. 使得GC压力更小，这样Java发生GC不会去移动`Direct buffer`了。





























