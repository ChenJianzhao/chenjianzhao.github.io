本章主要内容

- Channel
- ChannelHandler
- ChannePipeline
- ChannelHandlerContext

我们在上一章研究的 ByteBuf 是一个用来“包装”数据的容器。在本章我们将探讨这些容器是如何在应用程序中进行传输的，以及如何处理它们“包装”的数据。

Netty 在这方面提供了强大的支持。它让Channelhandler 链接在ChannelPipeline上使数据处理更加灵活和模块化。

在这一章中，下面我们会遇到各种各样 Channelhandler，ChannelPipeline 的使用案例，以及重要的相关的类Channelhandlercontext 。我们将展示如何将这些基本组成的框架可以帮助我们写干净可重用的处理实现。



## ChannelHandler 家族

在我们深入研究 ChannelHandler 内部之前，让我们花几分钟了解下这个 Netty 组件模型的基础。这里先对ChannelHandler 及其子类做个简单的介绍。

### Channel 生命周期

Channel 有个简单但强大的状态模型，与 ChannelInboundHandler API 密切相关。下面表格是 Channel 的四个状态

Table 6.1 Channel lifeycle states

| 状态                  | 描述                                    |
| ------------------- | ------------------------------------- |
| channelUnregistered | channel已创建但未注册到一个 EventLoop.          |
| channelRegistered   | channel 注册到一个 EventLoop.              |
| channelActive       | channel 变为活跃状态(连接到了远程主机)，现在可以接收和发送数据了 |
| channelInactive     | channel 处于非活跃状态，没有连接到远程主机             |

Channel 的正常的生命周期如下图，当状态出现变化，就会触发对应的事件，这样就能与 ChannelPipeline 中的 ChannelHandler进行及时的交互。

Figure 6.1 Channel State Model

![Figure_6.1](./channelHandler/Figure_6.1.jpg)



### ChannelHandler 生命周期

ChannelHandler 定义的生命周期操作如下表，当 ChannelHandler 添加到 ChannelPipeline，或者从 ChannelPipeline 移除后，对应的方法将会被调用。每个方法都传入了一个 ChannelHandlerContext 参数

Table 6.2 ChannelHandler lifecycle methods

| 类型              | 描述                                       |
| --------------- | ---------------------------------------- |
| handlerAdded    | 当 ChannelHandler 添加到 ChannelPipeline 调用  |
| handlerRemoved  | 当 ChannelHandler 从 ChannelPipeline 移除时调用 |
| exceptionCaught | 当 ChannelPipeline 执行抛出异常时调用              |



### ChannelHandler 子接口

Netty 提供2个重要的 ChannelHandler 子接口：

- ChannelInboundHandler - 处理进站数据和所有状态更改事件
- ChannelOutboundHandler - 处理出站数据，允许拦截各种操作



### ChannelHandler 适配器

Netty 提供了一个简单的 ChannelHandler 框架实现，给所有声明方法签名。这个类 `Chan  nelHandlerAdapter` 的方法,主要推送事件 到 pipeline 下个 ChannelHandler 直到 pipeline 的结束。这个类 也作为 ChannelInboundHandlerAdapter 和ChannelOutboundHandlerAdapter 的基础。所有三个适配器类的目的是作为自己的实现的起点;您可以扩展它们,覆盖你需要自定义的方法。



### ChannelInboundHandler

ChannelInboundHandler 的生命周期方法在下表中，当接收到数据或者与之关联的 Channel 状态改变时调用。之前已经注意到了，这些方法与 Channel 的生命周期接近

Table 6.3 ChannelInboundHandler methods

| 类型                        | 描述                                       |
| ------------------------- | ---------------------------------------- |
| channelRegistered         | 当Channel 已经注册到它的EventLoop 并且能够处理I/O 时被调用 |
| channelUnregistered       | 当Channel 从它的EventLoop 注销并且无法处理任何I/O 时被调用 |
| channelActive             | 当Channel 处于活动状态时被调用；Channel 已经连接/绑定并且已经就绪 |
| channelInactive           | 当Channel 离开活动状态并且不再连接它的远程节点时被调用          |
| channelReadComplete       | 当Channel上的一个读操作完成时被调用                    |
| channelRead               | 当从Channel 读取数据时被调用                       |
| channelWritabilityChanged | 当Channel 的可写状态发生改变时被调用。用户可以确保写操作不会完成得太快（以避免发生OutOfMemoryError）或者可以在Channel 变为再次可写时恢复写入。可以通过调用Channel 的isWritable()方法来检测Channel 的可写性。与可写性相关的阈值可以通过Channel.config().setWriteHighWaterMark()和Channel.config().setWriteLowWater-Mark()方法来设置 |
| userEventTriggered(...)   | 当ChannelnboundHandler.fireUserEventTriggered()方法被调用时被调用，因为一个POJO 被传经了ChannelPipeline |

注意，ChannelInboundHandler 实现覆盖了 channelRead() 方法处理进来的数据用来响应释放资源。Netty 在 ByteBuf 上使用了资源池，所以当执行释放资源时可以减少内存的消耗。



Listing 6.1 Handler to discard data

```java
@ChannelHandler.Sharable
public class DiscardHandler extends ChannelInboundHandlerAdapter {  //1.扩展 ChannelInboundHandlerAdapter

    @Override
    public void channelRead(ChannelHandlerContext ctx,
                                     Object msg) {
        ReferenceCountUtil.release(msg); 	//2.ReferenceCountUtil.release() 来丢弃收到的信息
    }

}
```



