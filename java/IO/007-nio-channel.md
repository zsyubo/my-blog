# 什么是Channel

`channel`被称为`通道`，类似于BIO的流，用于网络、文件等I/O操作。与传统流不同，`Channel`是全双工的(通信允许数据在两个方向上同时传输)。

`Channel`主要有以下4中：

1. `FileChannel`：用于文件I/O,只支持阻塞模式(因为数据就在本地文件中，固定的)。
2. `DatagramChannel`: 用于UDP通信。
3. `SocketChannel`: 用于TCP`客户端`的网络通信
4. `ServerSocketChannel`：用于TCP`服务端`的网络通信。

# `FileChannel`

文件通道，可以对文件的读写，但是文件通道是阻塞式的。

```java
public class NioFileCopy {
    public static void main(String[] args) throws IOException {
        FileInputStream inputStream = new FileInputStream("buffer.txt");
        FileOutputStream outputStream = new FileOutputStream("buffer2.txt");

        FileChannel input = inputStream.getChannel();
        FileChannel output= outputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate( 512 );
        while ( true ){
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
}
```

上面代码为一个简单的文件拷贝案例。我们可以从BIO的`InputStream`中获取到`Channel`。



# transferFrom和transferTo

`transferFrom`和`transferTo`允许我们将一个通道和另一个通道直接相连，这样我们就不用像普通IO编程那样，先读取到Buffer中再写入到其他通道，`省了一次拷贝过程`。

```java
   public static void main(String[] args) throws IOException {
        long start = System.currentTimeMillis();
        RandomAccessFile fromFile = new RandomAccessFile("/Users/huyuanfu/Downloads/spring-1.mp4", "rw");
        FileChannel fromChannel = fromFile.getChannel();
        RandomAccessFile toFile = new RandomAccessFile("/Users/huyuanfu/Downloads/spring-2.mp4", "rw");
        FileChannel toChannel = toFile.getChannel();
        System.out.println("拷贝字节："+ fromChannel.size() );
        toChannel.transferFrom(fromChannel,0, fromChannel.size()  );
//        fromChannel.transferTo(0, fromChannel.size() , toChannel );
        long end = System.currentTimeMillis();

        System.out.println("cost time:" + (end-start));
        ///Users/huyuanfu/Downloads

        //拷贝字节：1439928644
        //cost time:8064   8秒多
    }
```

`transferFrom`+堆外内存 可以说是java中的零拷贝。



RandomAccessFile(随机访问)

> 这里“随机”是指可以跳转到文件的任意位置处读写数据。在访问一个文件的时候，不必把文件从头读到尾，而是希望像访问一个数据库一样“随心所欲”地访问一个文件的某个部分，这时使用RandomAccessFile类就是最佳选择。































