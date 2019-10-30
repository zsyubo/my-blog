# Server

```java
 static Map<String, SocketChannel > channelMap = new HashMap<>();

    public static void main(String[] args) throws IOException {
      // 创建ServerSocketChannel 端口
        ServerSocketChannel  serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking( false );
        serverSocketChannel.bind( new InetSocketAddress(8899 ));
			// 打开Selector
        Selector selector = Selector.open();
      // 注册 serverSocketChannel 连接就绪事件 到 selector上。
        serverSocketChannel.register( selector, SelectionKey.OP_ACCEPT );
        while(true){
          // 阻塞，当有 事件发生时返回
            selector.select();
          // 获取事件set
            Set<SelectionKey > kes = selector.selectedKeys();
						// 迭代
            Iterator<SelectionKey> it = kes.iterator();
            while ( it.hasNext() ){
                SelectionKey selectionKey = it.next();
              // 如果是连接事件
                if ( selectionKey.isAcceptable() ){
                  	// 因为我们注册时就是注册的ServerSocketChannel，所以可以直接强转
                    ServerSocketChannel serverSocketChannel1 = (ServerSocketChannel) selectionKey.channel();
                  // 从 ServerSocketChannel 获取到一个SocketChannel(连接)
                    SocketChannel channel = serverSocketChannel.accept();
                  // 设置为非阻塞模式。
                    channel.configureBlocking( false );
                  // 将获取到的通道注册到selector上，同时注册 可读事件。
                    channel.register( selector, SelectionKey.OP_READ  );

                    String   key ="[" + UUID.randomUUID().toString()+ "]";
                    System.out.println("客户端："+ channel+ " 已连接！");
                    channelMap.put( key, channel );
                }else if(selectionKey.isReadable() ){
								//	 可读事件。
                  // 注册是就是注册的 SocketChannel，所以能直接强转。
                  SocketChannel channel = (SocketChannel) selectionKey.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate( 512 );
                  // 处理数据的逻辑。
                    while( true ){
                        byteBuffer.clear();
                        int read = channel.read( byteBuffer );
                        if ( read <= 0){
                            break;
                        }
                        byteBuffer.flip();
                        Charset charset =  Charset.forName("utf-8");
                        String msg = String.valueOf(charset.decode( byteBuffer ).array() );
                        String clientName ="";
                      // 在map中找到当前通过的id。
                        for (Map.Entry<String, SocketChannel >  kk: channelMap.entrySet()   ){
                            SocketChannel client = kk.getValue() ;
                            if ( channel ==  client ){
                                clientName= kk.getKey();
                            }
                        }
                        msg= clientName+ msg;
                      // 向各个通道写回数据。
                        for (Map.Entry<String, SocketChannel >  kk: channelMap.entrySet()   ){
                            SocketChannel client = kk.getValue() ;
                            if ( channel ==  client ){
                                continue;
                            }
                            ByteBuffer byteBuffer1 = ByteBuffer.allocate( 1024 );
                            byteBuffer1.put( msg.getBytes() );
                            byteBuffer1.flip();
                           client.write( byteBuffer1 );
                        }
                    }
                }
            }
          // 非常重要。
            it.remove();
        }
    }
```

这就是一个单线程案例，利用NIo，我们使用过单线程也能效率很高的模式。当然此Demo也有不足，比如断线的情况。

我们可以用使用`nc`命令来测试：`nc 127.0.0.1 8899`,然后直接输入信息回车就行。



# cleint

```java
 public static void main(String[] args) throws IOException {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking( false );
        socketChannel.connect( new InetSocketAddress("127.0.0.1", 8899));

        Selector selector = Selector.open();
        // 注意这里是 连接事件
        socketChannel.register( selector, SelectionKey.OP_CONNECT );

        while(true ){
            selector.select();
            Set<SelectionKey > key = selector.selectedKeys();
            key.forEach( selectionKey -> {
                if ( selectionKey.isConnectable() ){
                    // 连接成功事件
                    SocketChannel client = (SocketChannel) selectionKey.channel();
                    if ( client.isConnectionPending() ){
                        try {
                            client.finishConnect();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        ByteBuffer byteBuffer = ByteBuffer.allocate( 512 );
                        System.out.println("连接成功");
                        ExecutorService executorService = Executors.newSingleThreadExecutor(  Executors.defaultThreadFactory());
                        executorService.submit(new Runnable() {
                            @Override
                            public void run() {
                                while(true){
                                    byteBuffer.clear();
                                    InputStreamReader inputStreamReader = new InputStreamReader(  System.in );
                                    BufferedReader b = new BufferedReader( inputStreamReader );
                                    try {
                                        String msg = b.readLine();
                                        byteBuffer.put( msg.getBytes() );
                                        byteBuffer.flip();
                                        client.write( byteBuffer );

                                    } catch (IOException e) {
                                        e.printStackTrace();
                                    }
                                }
                            }
                        });
                    }
                    // 注册读取事件
                    try {
                        client.register( selector, SelectionKey.OP_READ );
                    } catch (ClosedChannelException e) {
                        e.printStackTrace();
                    }
                } else if( selectionKey.isReadable() ){
                    SocketChannel client = (SocketChannel) selectionKey.channel();
                    ByteBuffer byteBuffer = ByteBuffer.allocate( 512 );
                    while( true ) {
                        byteBuffer.clear();

                        int read = 0;
                        try {
                            read = client.read(byteBuffer);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        if (read <= 0) {
                            break;
                        }
                        byteBuffer.flip();
                        String msg = new String( byteBuffer.array(), 0, read );
                        System.out.println(msg);
                    }
                }
            });
            key.clear();
        }
    }
```

一样复杂。。。所以才需要Netty。