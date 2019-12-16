# Selector

称为多路选择器。它的作用是让我们用一个线程就可以监听和处理多个`SocketChannel`上的事件。如图：

<img src="https://s2.ax1x.com/2019/10/29/Kf30I0.png" height=300px width=700px />

如图所示，一个`Selector`通过`select()`，检查多个`Channel`上的注册事件，并把感兴趣的事件收集到集合中，并返回给工作线程，工作线程遍历事件集合，并判断是何事件，并进行相应的处理。

# 事件类型

创建一个`Selector`：`Selector selector = Selector.open();`

注册事件：`serverSocketChannel.register(selector, 事件值)`

有4种事件类型：

1. 连接事件`OP_CONNECT`
2. 准备就绪事件`OP_ACCEPT`
3. 读取事件`OP_READ`
4. 写入事件`OP_WRITE`































