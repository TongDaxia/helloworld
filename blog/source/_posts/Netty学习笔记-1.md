---
title: Netty学习笔记-1
date: 2019-10-21
tags: [java,网络编程] 
description: Netty是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器和客户端。
---



###### 阻塞IO示例



```
 ServerSocket serverSocket = new ServerSocket(8000);

        // (1) 接收新连接线程
        new Thread(() -> {
            while (true) {
                try {
                    // (1) 阻塞方法获取新的连接
                    Socket socket = serverSocket.accept();

                    // (2) 每一个新的连接都创建一个线程，负责读取数据
                    new Thread(() -> {
                        try {
                            byte[] data = new byte[1024];
                            InputStream inputStream = socket.getInputStream();
                            while (true) {
                                int len;
                                // (3) 按字节流方式读取数据
                                while ((len = inputStream.read(data)) != -1) {
                                    System.out.println(new String(data, 0, len));
                                }
                            }
                        } catch (IOException e) {
                        }
                    }).start();

                } catch (IOException e) {
                }

            }
        }).start();
```



```
public class IOClient {
    public static void main(String[] args) {
        new Thread(() -> {
            try {
                Socket socket = new Socket("127.0.0.1", 8000);
                while (true) {
                    try {
                        socket.getOutputStream().write((new Date() + ": hello world").getBytes());
                        socket.getOutputStream().flush();
                        Thread.sleep(2000);
                    } catch (Exception e) {
                    }
                }
            } catch (IOException e) {
            }
        }).start();
    }
}

```

弊端：

在传统的IO模型中，每个连接创建成功之后都需要一个线程来维护，每个线程包含一个while死循环，那么1w个连接对应1w个线程。具体问题如下：

1. 线程资源受限：线程是操作系统中非常宝贵的资源，同一时刻有大量的线程处于阻塞状态是非常严重的资源浪费，操作系统耗不起
2. 线程切换效率低下：单机cpu核数固定，线程爆炸之后操作系统频繁进行线程切换，应用性能急剧下降。
3. 除了以上两个问题，IO编程中，我们看到数据读写是以字节流为单位，效率不高。



###### NIO

```
 private final int port = 8787;
    private final int BLOCK_SIZE = 4096;

    private Selector selector;

    private ByteBuffer receiveBuffer = ByteBuffer.allocate(BLOCK_SIZE);
    private ByteBuffer sendBuffer = ByteBuffer.allocate(BLOCK_SIZE);




    //构造函数
    public NioServer() throws IOException {
        //打开服务器套接字通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //服务器配置为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //获取与通道关联的 ServerSocket对象
        ServerSocket serverSocket = serverSocketChannel.socket();
        //绑定端口
        serverSocket.bind(new InetSocketAddress(port));
        //打开一个选择器
        selector = Selector.open();
        //注册到selector上，等待连接
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        System.out.println("Server:init successfuly.");
    }

    /**
     * 监听端口
     */
    private void linstenr() throws Exception{

        while (true) {
            //选择一组键
            selector.select();
            //返回获取选择的键集
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            if(selectionKeys.isEmpty()){
                continue;
            }
            //遍历，循环处理请求的键集
            Iterator<SelectionKey> iterator =  selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey selectionKey = (SelectionKey) iterator.next();
                iterator.remove();
                handlerKey(selectionKey);
            }

            Thread.sleep(400);
        }

    }

    /**
     * 处理对应的  SelectionKey
     * @param selectionKey
     */
    private void handlerKey(SelectionKey selectionKey) throws IOException{

        ServerSocketChannel server;
        SocketChannel client;

        // 测试此键的通道是否已准备好接受新的套接字连接
        if(selectionKey.isAcceptable()){
            //此键对应的关联通道
            server = (ServerSocketChannel)selectionKey.channel();
            //接受到此通道套接字的连接
            client = server.accept();
            //配置为非阻塞
            client.configureBlocking(false);
            //注册到selector 等待连接
            client.register(selector, SelectionKey.OP_READ);

        }

        else if (selectionKey.isReadable()) {

            client = (SocketChannel)selectionKey.channel();
            //将缓冲区清空，下面读取
            receiveBuffer.clear();
            //将客户端发送来的数据读取到 buffer中
            int count = client.read(receiveBuffer);
            if(count >0){
                String receiveMessage = new String(receiveBuffer.array(),0,count);
                System.out.println("Server:接受客户端的数据:" + receiveMessage);
                client.register(selector, SelectionKey.OP_WRITE);
            }
        }

        else if (selectionKey.isWritable()) {
            //发送消息buffer 清空
            sendBuffer.clear();
            //返回该键对应的通道
            client = (SocketChannel)selectionKey.channel();
            String sendMessage = "Send form Server...Hello... "+new Random().nextInt(100)+" .";
            //向缓冲区中写入数据
            sendBuffer.put(sendMessage.getBytes());
            //put了数据，标志位被改变
            sendBuffer.flip();
            //数据输出到通道
            client.write(sendBuffer);
            System.out.println("Server:服务器向客户端发送数据:" + sendMessage);
            client.register(selector, SelectionKey.OP_READ);
        }
    }
    public static void main(String[] args) {
        try {
            NioServer nioServer = new NioServer();
            nioServer.linstenr();
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }


```



```

    private static final int BLOCK_SIZE = 4096;
    private static ByteBuffer sendBuffer = ByteBuffer.allocate(BLOCK_SIZE);

    private static ByteBuffer receiveBuffer = ByteBuffer.allocate(BLOCK_SIZE);

    private static final InetSocketAddress SERVER_ADDRESS = new InetSocketAddress("127.0.0.1",8787);


    public static void main(String[] args) {

        try {
            //打开socket通道
            SocketChannel socketChannel = SocketChannel.open();
            //设置为非阻塞模式
            socketChannel.configureBlocking(false);
            //打开选择器
            Selector selector = Selector.open();
            //向selector 选择器注册此通道
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
            //链接
            socketChannel.connect(SERVER_ADDRESS);

            SocketChannel client;
            while (true) {
                //选择一组键
                selector.select();
                //返回此选择器的已选择键集
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = selectionKeys.iterator();
                //遍历对应的 SelectionKey 处理
                while (iterator.hasNext()) {
                    SelectionKey selectionKey = (SelectionKey) iterator.next();
                    //判断此键的通道是否已完成其套接字连接操作
                    if (selectionKey.isConnectable()) {
                        System.out.println("Client:  already connected.");
                        client = (SocketChannel)selectionKey.channel();
                        //判断该通道是否进行连接过程、完成连接过程
                        if(client.isConnectionPending()){
                            client.finishConnect();

                            sendBuffer.clear();
                            sendBuffer.put("hello nio server".getBytes());
                            sendBuffer.flip();

                            client.write(sendBuffer); //将数据写入该通道
                            client.register(selector, SelectionKey.OP_READ);
                        }
                    }
                    else if(selectionKey.isReadable()){
                        //获取该键中对应的通道
                        client = (SocketChannel)selectionKey.channel();

                        receiveBuffer.clear();
                        int count = client.read(receiveBuffer);
                        if(count > 0){
                            String receiveMessage = new String(receiveBuffer.array(),0,count);
                            System.out.println("Client：接收到来自Server的消息，" + receiveMessage);
                            client.register(selector, SelectionKey.OP_WRITE);
                        }
                    }
                    else if(selectionKey.isWritable()){
                        sendBuffer.clear();
                        client = (SocketChannel) selectionKey.channel();
                        String sendText = "hello server,key..";
                        sendBuffer.put(sendText.getBytes());
                        //将缓冲区各标志复位,因为向里面put了数据标志被改变要想从中读取数据发向服务器,就要复位
                        sendBuffer.flip();
                        client.write(sendBuffer);
                        System.out.println("Client:客户端向服务器端发送数据--："+sendText);
                        client.register(selector, SelectionKey.OP_READ);
                    }
                }
                selectionKeys.clear();
                Thread.sleep(300);
            }
        } catch (Exception e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
```



或者采用下面的代码:

```
 public static void main(String[] args) throws IOException {
        Selector serverSelector = Selector.open();
        Selector clientSelector = Selector.open();

        new Thread(() -> {
            try {
                // 对应IO编程中服务端启动
                ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                listenerChannel.socket().bind(new InetSocketAddress(8000));
                listenerChannel.configureBlocking(false);
                listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                while (true) {
                    // 监测是否有新的连接，这里的1指的是阻塞的时间为1ms
                    if (serverSelector.select(1) > 0) {
                        Set<SelectionKey> set = serverSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();

                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();

                            if (key.isAcceptable()) {
                                try {
                                    // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                    SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                    clientChannel.configureBlocking(false);
                                    clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                } finally {
                                    keyIterator.remove();
                                }
                            }

                        }
                    }
                }
            } catch (IOException ignored) {
            }

        }).start();

        new Thread(() -> {
            try {
                while (true) {
                    // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为1ms
                    if (clientSelector.select(1) > 0) {
                        Set<SelectionKey> set = clientSelector.selectedKeys();
                        Iterator<SelectionKey> keyIterator = set.iterator();

                        while (keyIterator.hasNext()) {
                            SelectionKey key = keyIterator.next();

                            if (key.isReadable()) {
                                try {
                                    SocketChannel clientChannel = (SocketChannel) key.channel();
                                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                    // (3) 读取数据以块为单位批量读取
                                    clientChannel.read(byteBuffer);
                                    byteBuffer.flip();
                                    System.out.println(Charset.defaultCharset().newDecoder().decode(byteBuffer)
                                            .toString());
                                } finally {
                                    keyIterator.remove();
                                    key.interestOps(SelectionKey.OP_READ);
                                }
                            }

                        }
                    }
                }
            } catch (IOException ignored) {
            }
        }).start();


    }
```



###### NettyDemo



```
public final class SimpleServer {

    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .handler(new SimpleServerHandler())
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                        }
                    });

            ChannelFuture f = b.bind(8888).sync();

            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    private static class SimpleServerHandler extends ChannelInboundHandlerAdapter {
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception {
            System.out.println("channelActive");
        }

        @Override
        public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
            System.out.println("channelRegistered");
        }

        @Override
        public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
            System.out.println("handlerAdded");
        }
    }
}

```







- Channel
- ChannelConfig
- ChannelId
- Unsafe
- Pipeline
- ChannelHander





1.设置启动类参数，最重要的就是设置channel 

2.创建server对应的channel，创建各大组件，包括ChannelConfig,ChannelId,ChannelPipeline,ChannelHandler,Unsafe等 

3.初始化server对应的channel，设置一些attr，option，以及设置子channel的attr，option，给server的channel添加新channel接入器，并出发addHandler,register等事件 

4.调用到jdk底层做端口绑定，并触发active事件，active触发的时候，真正做服务端口绑定



























