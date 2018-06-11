# Netty实战

## 第一部分 Netty的概念及体系结构

### 第1章 Netty-异步和事件驱动

#### 1.1 Java网络编程

 ![](http://static.zybuluo.com/vermouth9/aq2r7p8ss8fdzbiyzk3oqx20/image.png)

* ServerSocket上的```accept()```会一直阻塞到一个连接建立，随后返回一个新的Socket用于客户端和服务器之间的通信。该ServerSocket将继续监听传入的连接。
* BufferedReader和PrintWriter都衍生自Socket的输入输出流，前者从一个字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流。
* ```readLine()```方法将会阻塞，直到在3处一个由换行符或者回车符结尾的字符串被读取。

这个代码片段只能同时处理一个连接，要管理多个并发客户端，**需要为每个新的客户端socket创建一个新的Thread**。

![](http://static.zybuluo.com/vermouth9/eoxk7rpsucy29pyglhfg32qm/image.png)

这种方法的缺点：

1. 任何时候都可能有大量线程处于休眠状态，等待输入或者输出数据就绪。
2. 需要为每个线程的调用栈分配内存。
3. 并发数特别大的时候，线程上下文切换带来很大的开销。

##### 1.1.1 Java NIO

位于JDK1.4 的```java.nio```包中

##### 1.1.2 选择器

使用事件通知API以确定在一组非阻塞套接字中有哪些已经就绪能够进行I/O相关的操作。因为可以在任何的时间检查任意的读操作或写操作的完成状态，所以**一个单一的线程便可以处理多个并发的连接**。

![](http://static.zybuluo.com/vermouth9/3y5pk2ltil99uebcjcwwcil2/image.png)

这种模型的好处：

1. 使用较少的线程便可以处理许多连接。
2. 当没有I/O操作需要处理的时候，线程也可以被用于其他任务。

#### 1.3 Netty的核心组件

##### 1.3.1 Channel

可以把```Channel```看作是传入或者传出数据的载体。

##### 1.3.2 回调

Netty在内部使用了回调来处理事件：当一个回调被触发时，相关的事件可以被一个接口：```ChannelHandler```的实现处理。

##### 1.3.3 Future

Future提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符：它将在未来的某个时刻完成，并提供对其结果的访问。

JDK内置的```Future```提供的实现只允许手动检查对应的操作是否完成，或者一直阻塞直到完成，这是非常繁琐的，所有Netty提供了自己的实现：```ChannelFuture```，用于在执行异步操作的时候使用。

```ChannelFuture```提供了几种额外的方法，能够注册一个或多个```ChannelFutureListener```实例，监听器（```ChannelFutureListener```）的回调方法```operationComplete()```将会在对应的操作完成时被调用。监听器可以判断操作是成功还是失败，如果失败，可以检索产生的Throwable。即```ChannelFutrueListener```消除了手动检查对应的操作是否完成的必要。

每个Netty的出站I/O操作都将返回一个ChannelFuture，也就是说它们都不会阻塞。

```java
Chanel channel = ...;
//connect()方法将会直接返回，而不会阻塞。
ChannelFuture future = channel.connect(new InetSocketAddress("192.168.0.1", 25));

//注册一个ChannelFutureListener，以便在操作完成时获得通知
future.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture future) {
        if (future.isSuccess()) {
            ByteBuf buffer = Unpooled.copiedBuffer("Hello", Charset.defaultCharset());
            //将数据异步地发送到远程节点，返回一个ChannelFuture
            ChannelFuture wf = future.channel().writeAndFlush(buffer);
            ...
        }else {
            //如果发生错误，则访问描述原因的Throwable
            Throwable cause = future.cause();
            cause.printStackTrace();
        }
    }
});
```

***回调和Future是相互补充的机制。***

##### 1.3.4 事件和ChannelHandler

Netty提供了大量开箱即用的ChannelHandler实现，包括用于各种协议（如HTTP和SSL/TLS）的ChannelHandler。

### 第2章 你的第一款Netty应用程序

#### 2.3 编写Echo服务器

##### 2.3.1 ChannelHandler和业务逻辑

```ChannelHandler```是一个接口族的父接口，它的实现负责接收并响应事件通知。

因为Echo服务器会响应传入的消息，所有需要实现```ChannelInboundHandler```接口（继承了```ChannelHandler接口```），用来定义响应入站事件的方法。这个简单的应用程序可以直接继承```ChannelInboundHandlerAdapter```类就足够了（提供了```ChannelInboundHandler```的默认实现）。

```java
@ChannelHandler.Sharable            //标示一个ChannelHandler可以被多个Channel安全地共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    /*
     * 将接收到的消息写给发送者，而不冲刷出站消息
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf in = (ByteBuf)msg;
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));
        ctx.write(in);
    }
    /*
    将未决消息冲刷到远程节点，并且关闭该Channel
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }
	/*
    允许对Throwable的任何子类型做出反应。
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

记住以下这些关键点：

* 针对不同类型的事件来调用```ChannelHandler```
* 应用程序通过实现或者扩展```ChannelHandler```来挂钩到事件的生命周期，并且提供自定义的应用程序逻辑。
* 在架构上，```ChannelHandler```有助于保持业务逻辑与网络处理代码的分离。

##### 2.3.2 引导服务器

* 绑定到服务器将在其上监听并接受传入连接请求的端口；
* 配置Channel，以将有关的入站消息通知给```EchoServerHandler```实例。

```java
public class EchoServer {
    private final int port;

    EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        new EchoServer(12345).start();
    }

    private void start() throws Exception {
        EchoServerHandler serverHandler = new EchoServerHandler();
        //1. 创建EventLoopGroup
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //2. 创建ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            b.group(group)
                    //3. 指定所使用的NIO传输Channel
                    .channel(NioServerSocketChannel.class)
                    //4. 使用指定的端口设置套接字地址
                    .localAddress(new InetSocketAddress(port))
                    //5. 添加一个EchoServerHandler到子Channel的ChannelPipeline
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        //当一个新的连接被接受时，一个新的子Channel将会被创建，
                        //而ChannelInitializer将会把一个你的EchoServerHandler的实例添加到该Channel的ChannelPipeline中。
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //EchoServerHandler被标注为@Sharable，所以我们可以总是使用同样的实例
                            socketChannel.pipeline().addLast(serverHandler);
                        }
                    });
            //6. 异步地绑定服务器，调用sync()方法阻塞等待直到绑定完成
            ChannelFuture future = b.bind().sync();
            //7. 获取Channel的CloseFuture，并且阻塞当前直到服务器的Channel关闭
            future.channel().closeFuture().sync();
        }finally {
            //8. 关闭EventLoopGroup，释放所有的资源
            group.shutdownGracefully().sync();
        }
    }
}
```

#### 2.4 编写Echo客户端

Echo客户端将会：

1. 连接到服务器；
2. 发送一个或者多个消息；
3. 对于每个消息，等待并接收从服务器发回的相同的消息；
4. 关闭连接。

##### 2.4.1 通过ChannelHandler实现客户端逻辑

如同服务器，客户端将有一个用来处理数据的```ChannelInboundHandler```，在这个场景中，可以继承```SimpleChannelInboundHandler```。

```java
@ChannelHandler.Sharable        //标记该类的实例可以被多个Channel共享
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {
    //在到服务器的连接已经建立之后将被调用
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!", CharsetUtil.UTF_8));
    }
    //当从服务器接收到一条消息时被调用
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf byteBuf) throws Exception {
        System.out.println("client receiver: " + byteBuf.toString(CharsetUtil.UTF_8));
    }
    //在处理过程中引发异常时被调用
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

**注**：由服务器发送的消息可能会被分块接收。也就是说，如果服务器发送了5字节，那么不能保证这5字节会被一次性接收。

##### 2.4.2 引导客户端

客户端使用主机和端口参数来连接远程地址。

```java
public class EchoClient {
    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        new EchoClient("localhost", 12345).start();
    }

    private void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
                    .channel(NioSocketChannel.class)
                    .remoteAddress(new InetSocketAddress(host, port))
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new EchoClientHandler());
                        }
                    });
            ChannelFuture future = b.connect().sync();
            future.channel().closeFuture().sync();
        }finally {
            group.shutdownGracefully().sync();
        }
    }
}
```

#### 2.5 构建和运行Echo服务器和客户端

发生的事：

1. 一旦客户端建立连接，它就发送它的消息：Netty rocks!；
2. 服务器报告接收到的消息，并将其会送给客户端；
3. 客户端报告返回的消息并退出。



