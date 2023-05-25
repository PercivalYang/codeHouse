
- [Netty](#netty)
  - [EventLoop](#eventloop)
  - [Channel](#channel)
    - [ChannelFuture](#channelfuture)
  - [Future \&\& Promise](#future--promise)
  - [Handler \&\& Pipeline](#handler--pipeline)
  - [ByteBuf](#bytebuf)
    - [直接内存 \&\& 堆内存](#直接内存--堆内存)
    - [池化](#池化)
    - [slice](#slice)
    - [copy \& duplicate \& CompositeByteBuf](#copy--duplicate--compositebytebuf)
    - [工具类Unpooled](#工具类unpooled)

# Netty

一张服务器建立监听端口，客户端发起连接并发送字符串数据的流程图：

![20230506212934](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230506212934.png)

## EventLoop

EventLoop本质是一个单线程执行器，可用于处理连接、接收数据、发送数据等事件。

EventLoopGroup是一组EventLoop，通常Channel会选择EventLoopGroup中的一个EventLoop进行注册，以便处理连接、接收数据、发送数据等事件，保证了线程的安全性

`NioEventLoopGroup`: 可用于处理io事件、普通任务和定时任务

`DefaultEventLoopGroup`: 可用于处理普通任务和定时任务(不能处理io事件)

举个例子，服务器的代码如下：

```java
// 2个可以处理普通任务和定时任务的EventLoop
DefaultEventLoopGroup normalWorkers = new DefaultEventLoopGroup(2);
new ServerBootstrap()
    // 3个可以处理io、普通任务和定时任务的EventLoop
    .group(new NioEventLoopGroup(1), new NioEventLoopGroup(2))
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        @Override
        protected void initChannel(NioSocketChannel ch)  {
            ch.pipeline().addLast(new LoggingHandler(LogLevel.DEBUG));
            ch.pipeline().addLast(normalWorkers,"myhandler",
              new ChannelInboundHandlerAdapter() {
                @Override
                public void channelRead(ChannelHandlerContext ctx, Object msg) {
                    ByteBuf byteBuf = msg instanceof ByteBuf ? ((ByteBuf) msg) : null;
                    if (byteBuf != null) {
                        byte[] buf = new byte[16];
                        ByteBuf len = byteBuf.readBytes(buf, 0, byteBuf.readableBytes());
                        log.debug(new String(buf));
                    }
                }
            });
        }
    }).bind(8080).sync();
```

- 为什么要单独创建一个`DefaultEventLoopGroup`?

  A: `NioEventLoop`不止负责io事件还要负责普通任务时，如果普通任务执行时间过长，为阻塞其绑定的`Channel`上之后到来的数据的io事件。因此通常`NioEventLoop`只负责处理io事件，普通任务和定时任务交给`DefaultEventLoop`处理。(本质上是将IO事件和普通任务交由不同的线程处理)

- 不同Handler执行结束后如何切换`EventLoop`？

  A: `io.netty.channel.AbstractChannelHandlerContext`抽象类中存在各种`invoke*`的方法，包括`invokeRead`, `invokeWrite`, `invokeHandler`等。抽象来说，在方法内部会判断Handler是否绑定的同一个`EventLoop`，如果绑定的话就直接在EventLoop本地执行Handler，否则新建一个`EventLoop`(Thread)来执行。下面以源码中的`invokeChannelRead`为例

  ![20230517223619](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230517223619.png)

## Channel

channel的常用方法：

- `close()`: 关闭channel
- `closeFuture()`: 返回一个`ChannelFuture`，用于监听channel的关闭事件
- `pipeline()`: 返回一个`ChannelPipeline`，用于管理channel中的handler
- `writeAndFlush()`: 将数据写入channel并刷新
- `write()`: 将数据写入channel

### ChannelFuture

`BootStrap`的`connect()`方法返回一个`ChannelFuture`，我们可以利用`ChannelFuture`对象来监听**连接建立成功后的事件**

```java
public ChannelFuture connect(String inetHost, int inetPort) {
    return connect(InetSocketAddress.createUnresolved(inetHost, inetPort));
}
```

`ChannelFuture`对象可以获取到建立连接后传输数据的`Channel`对象，有两种方法获取：

- `sync()`方法同步获取
- `addListener()`方法异步获取

```java
// 同步获取channel
Channel channel = channelFuture.sync().channel();
// 异步获取
channelFuture.addListener((ChannelFutureListener) future -> {
  // 内部是获取成功
  System.out.println(future.channel())
});
```

同样`channel`也有`closeFuture()`方法，用于监听channel的关闭事件，例如

```java
ChannelFuture closeFuture = channel.clouseFuture()
closeFuture.addListener(new ChannelFutureListener() {
    @Override
    public void operationComplete(ChannelFuture future) throws Exception {
        log.debug("处理关闭之后的操作");
        group.shutdownGracefully();
    }
});
```

## Future && Promise

Netty的`Future`继承自`java.util.concurrent.Future`，JDK中的`Future`只能同步等待任务结束才能得到结果，而Netty的`Future`可以异步获取任务结果(例如前面提到的`addListener`)。

`Promise`继承自Netty的`Future`，它有`Future`的所有功能，同时独立于任务之外单独存在，作为线程之间传递任务结果的容器。

## Handler && Pipeline

ChannelHandler用于处理Channel上的各种事件，分**入站**、**出站**两种。

- 入站：`ChannelInboundHandler`，主要用于读取客户端数据，并返回结果
- 出站：`ChannelOutboundHandler`，对返回结果进行加工

ChannelPipeline是一个双向链表，链表中默认会有`head`和`tail`分别代表头尾。**入站的Handler顺序是从`head`到`tail`，出站的Handler顺序是从`tail`到`head`。**

## ByteBuf

ByteBuf组成中有四个成员属性：

- readIndex
- writeIndex
- capacity
- max capacity

![20230521134353](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230521134353.png)

### 直接内存 && 堆内存

ByteBuf的申请有两种方式，一种是申请堆内存，一种是申请直接内存。直接内存相比堆内存的优势在于不需要拷贝到堆内存中再进行io操作(即读写效率高，"零拷贝")，但是直接内存的申请和释放需要调用系统调用，相比堆内存的申请和释放效率较低。

同时直接内存不受GC管理，对GC压力小，但是需要手动释放内存。

### 池化

池化让ByteBuf可以重用(类似线程池)，减少内存申请和释放的开销。

**retain & release**

Netty通过[“引用计数法”](./JVM.md#引用计数法)来控制回收内存，即每个ByeBuf都实现了`ReferenceCounted`接口

- ByteBuf的初始计数为1
- 调用`retain()`方法，计数加1
- 调用`release()`方法，计数减1
- 当计数为0时，底层内存会被回收

因为前面讲过pipeline，会将ByteBuf传递给下一个Handler，一般谁是最后的使用者，谁负责release

- Handler不处理ByteBuf直接通过`ctx.fireChannelRead(msg)`向后传递时，无需 release
- Handler将ByteBuf转换成Java对象，这时ByteBuf不再需要时，需 release
- 如果不再向后传递或者抛出异常，需 release
- pipeline有tail节点，会通过`TailContext`进行回收(这是对于入站消息)
- 同样对于出站消息，pipeline有head节点，会通过`HeadContext`进行回收

### slice

slice是将原始ByteBuf切片成多个ByteBuf，切片后的ByteBuf与原始ByteBuf**共享底层内存**，但是各自维护自己的readIndex、writeIndex和capacity。例如：

```java
ByteBuf buf = Unpooled.buffer(10);
buf.writeBytes("hello world".getBytes());
// 0是readIndex，5是length
ByteBuf slice = buf.slice(0, 5);
// 无参的话，切片readIndex到writeIndex的内容 
ByteBuf slice2 = buf.slice();
```

- 切片会将maxCapacity设置为原始ByteBuf的readIndex到writeIndex的长度，即切片后的ByteBuf不能再往里面写数据，否则会抛出IndexOutOfBoundsException

### copy & duplicate & CompositeByteBuf

`duplicate`与原始ByteBuf共享底层内存，但是各自维护自己的readIndex、writeIndex和capacity。

`copy`与原始ByteBuf不共享底层内存

`CompositeByteBuf`是一个复合ByteBuf，它可以将多个ByteBuf组合成一个ByteBuf，组合后的ByteBuf与原始ByteBuf**不共享底层内存**

### 工具类Unpooled

- `wrappedBuffer`：将字节数组、字节缓冲区、字符串包装成ByteBuf
