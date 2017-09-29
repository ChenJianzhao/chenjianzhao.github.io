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

Netty 提供了一个简单的 ChannelHandler 框架实现，给所有声明方法签名。

这个类 `ChannelHandlerAdapter` 的方法,主要推送事件 到 pipeline 下个 ChannelHandler 直到 pipeline 的结束。这个类 也作为 ChannelInboundHandlerAdapter 和ChannelOutboundHandlerAdapter 的基础。所有三个适配器类的目的是作为自己的实现的起点;您可以扩展它们,覆盖你需要自定义的方法。



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



Netty 用一个 WARN-level 日志条目记录未释放的资源,使其能相当简单地找到代码中的违规实例。然而,由于手工管理资源会很繁琐,您可以通过使用 SimpleChannelInboundHandler 简化问题。如下：



Listing 6.2 Handler to discard data

```java
@ChannelHandler.Sharable
public class SimpleDiscardHandler extends SimpleChannelInboundHandler<Object> {  //1.扩展 SimpleChannelInboundHandler

    @Override
    public void channelRead0(ChannelHandlerContext ctx,
                                     Object msg) {
        // No need to do anything special //2.不需做特别的释放资源的动作
    }

}
```

注意 **SimpleChannelInboundHandler** 会自动释放资源，而无需存储任何信息的引用。



### ChannelOutboundHandler

ChannelOutboundHandler 提供了出站操作时调用的方法。这些方法会被 Channel, ChannelPipeline, 和 ChannelHandlerContext 调用。

ChannelOutboundHandler 另个一个强大的方面是它具有在请求时延迟操作或者事件的能力。比如，当你在写数据到 remote peer 的过程中被意外暂停，你可以延迟执行刷新操作，然后在迟些时候继续。

下面显示了 ChannelOutboundHandler 的方法（继承自 ChannelHandler 未列出来）

Table 6.4 ChannelOutboundHandler methods

| 类型         | 描述                              |
| ---------- | ------------------------------- |
| bind       | 当请求将Channel 绑定到本地地址时被调用         |
| connect    | 当请求将Channel 连接到远程节点时被调用         |
| disconnect | 当请求将Channel 从远程节点断开时被调用         |
| close      | 当请求关闭Channel 时被调用               |
| deregister | 当请求将Channel 从它的EventLoop 注销时被调用 |
| read       | 当请求从Channel 读取更多的数据时被调用         |
| flush      | 当请求通过Channel 将入队数据冲刷到远程节点时被调用   |
| write      | 当请求通过Channel 将数据写到远程节点时被调用      |

几乎所有的方法都将 ChannelPromise 作为参数,一旦请求结束要通过 ChannelPipeline 转发的时候，必须通知此参数。



**ChannelPromise vs. ChannelFuture**

ChannelPromise 是 特殊的 ChannelFuture，允许你的 ChannelPromise 及其 操作 成功或失败。所以任何时候调用例如 Channel.write(...) 一个新的 ChannelPromise将会创建并且通过 ChannelPipeline传递。这次写操作本身将会返回 ChannelFuture， 这样只允许你得到一次操作完成的通知。Netty 本身使用 ChannelPromise 作为返回的 ChannelFuture 的通知，事实上在大多数时候就是 ChannelPromise 自身（ChannelPromise 扩展了 ChannelFuture）

如前所述,ChannelOutboundHandlerAdapter 提供了一个实现了 ChannelOutboundHandler 所有基本方法的实现的框架。 这些简单事件转发到下一个 ChannelOutboundHandler 管道通过调用 ChannelHandlerContext 相关的等效方法。你可以根据需要自己实现想要的方法。



### 资源管理

- 每当通过调用ChannelInboundHandler.channelRead()或者ChannelOutbound-Handler.write()方法来处理数据时，你都需要确保没有任何的资源泄漏。你可能还记得在前面的章节中所提到的，Netty 使用引用计数来处理池化的ByteBuf。所以在完全使用完某个ByteBuf 后，调整其引用计数是很重要的。


- 为了帮助你诊断潜在的（资源泄漏）问题，Netty提供了class ResourceLeakDetector，它将对你应用程序的缓冲区分配做大约1%的采样来检测内存泄露。相关的开销是非常小的。

  如果检测到了内存泄露，将会产生类似于下面的日志消息：

```
LEAK: ByteBuf.release() was not called before it's garbage-collected. Enable
advanced leak reporting to find out where the leak occurred. To enable
advanced leak reporting, specify the JVM option
'-Dio.netty.leakDetectionLevel=ADVANCED' or call
Resourc eLeakDetector.setLevel().
```



Netty 目前定义了4 种泄漏检测级别，如表6-5 所示。

Table 6.5 Leak detection levels

| 级别描 述    |                                          |
| -------- | ---------------------------------------- |
| Disables | 禁用泄漏检测。只有在详尽的测试之后才应设置为这个值                |
| SIMPLE   | 使用1%的默认采样率检测并报告任何发现的泄露。这是默认级别，适合绝大部分的情况  |
| ADVANCED | 使用默认的采样率，报告所发现的任何的泄露以及对应的消息被访问的位置        |
| PARANOID | 类似于ADVANCED，但是其将会对每次（对消息的）访问都进行采样。这对性能将会有很大的影响，应该只在调试阶段使用 |


泄露检测级别可以通过将下面的Java 系统属性设置为表中的一个值来定义：

```
java -D io.netty.leakDetectionLevel=ADVANCED
```



这样，我们就能在 ChannelInboundHandler.channelRead(...) 和 ChannelOutboundHandler.write(...) 避免泄漏。

当你处理 channelRead(...) 操作，并在消费消息(不是通过 ChannelHandlerContext.fireChannelRead(...) 来传递它到下个 ChannelInboundHandler) 时，要释放它，如下：

Listing 6.3 Handler that consume inbound data

```java
@ChannelHandler.Sharable
public class DiscardInboundHandler extends ChannelInboundHandlerAdapter {  
	//1.继承 ChannelInboundHandlerAdapter

    @Override
    public void channelRead(ChannelHandlerContext ctx,
                                     Object msg) {
        ReferenceCountUtil.release(msg); 
        //2使用 ReferenceCountUtil.release(...) 来释放资源
    }

}
```

**所以记得，每次处理消息时，都要释放它。**



**SimpleChannelInboundHandler -消费入站消息更容易**

使用入站数据和释放它是一项常见的任务，Netty 为你提供了一个特殊的称为 SimpleChannelInboundHandler 的 ChannelInboundHandler 的实现。**该实现将自动释放一个消息，一旦这个消息被用户通过channelRead0() 方法消费。**



当你在处理写操作，并丢弃消息时，你需要释放它。现在让我们看下实际是如何操作的。

Listing 6.4 Handler to discard outbound data

```java
@ChannelHandler.Sharable 
  public class DiscardOutboundHandler extends ChannelOutboundHandlerAdapter { 
  //1.继承 ChannelOutboundHandlerAdapter
  @Override
  public void write(ChannelHandlerContext ctx,
                                   Object msg, ChannelPromise promise) {
      ReferenceCountUtil.release(msg);  //2.使用 ReferenceCountUtil.release(...) 来释放资源
      promise.setSuccess();    		//3.通知 ChannelPromise 数据已经被处理

  }
}
```

重要的是，释放资源并通知 ChannelPromise。如果，ChannelPromise 没有被通知到，这可能会引发 ChannelFutureListener 不会被处理的消息通知的状况。

所以，总结下：

- 如果消息是被 `消耗/丢弃` ， 并不会被传入下个 ChannelPipeline 的 ChannelOutboundHandler ，调用ReferenceCountUtil.release(message) 。
- 一旦消息经过实际的传输，在消息被写或者 Channel 关闭时，它将会自动释放。




## ChannelPipeline

ChannelPipeline 是一系列的 ChannelHandler 实例，用于拦截 流经一个 Channel 的入站和出站事件,ChannelPipeline允许用户自己定义对入站/出站事件的处理逻辑，以及pipeline里的各个Handler之间的交互。

每一次创建了新的Channel ,都会新建一个新的 ChannelPipeline并绑定到Channel上。这个关联是 永久性的;Channel 既不能附上另一个 ChannelPipeline 也不能分离当前这个。这些都由Netty负责完成,而无需开发人员的特别处理。

根据它的起源,一个事件将由 ChannelInboundHandler 或 ChannelOutboundHandler 处理。随后它将调用 ChannelHandlerContext 实现转发到下一个相同的超类型的处理程序。



**ChannelHandlerContext**

一个 ChannelHandlerContext 使 ChannelHandler 与 ChannelPipeline 和 其他处理程序交互。**一个处理程序可以通知下一个 ChannelPipeline 中的 ChannelHandler 甚至动态修改 ChannelPipeline 的归属。**

下图展示了用于入站和出站 ChannelHandler 的 典型 ChannelPipeline 布局。

![Figure_6.2](./channelHandler/Figure_6.2.jpg)

Figure 6.2 ChannelPipeline and ChannelHandlers



上图说明了 ChannelPipeline 主要是一系列 ChannelHandler。通过ChannelPipeline ChannelPipeline 还提供了方法传播事件本身。如果一个入站事件被触发，它将被传递的从 ChannelPipeline 开始到结束。举个例子,在这个图中出站 I/O 事件将从 ChannelPipeline 右端开始一直处理到左边。

**ChannelPipeline 相对论**

你可能会说,从 ChannelPipeline 事件传递的角度来看,ChannelPipeline 的“开始” 取决于是否入站或出站事件。然而,Netty 总是指 ChannelPipeline 入站口(图中的左边)为“开始”,出站口(右边)作为“结束”。当我们完成使用 ChannelPipeline.add*() 添加混合入站和出站处理程序,每个 ChannelHandler 的“顺序”是它的地位从“开始”到“结束”正如我们刚才定义的。因此,如果我们在图6.1处理程序按顺序从左到右第一个ChannelHandler被一个入站事件将是#1,第一个处理程序被出站事件将是#5

随着管道传播事件,它决定下个 ChannelHandler 是否是相匹配的方向运动的类型。如果没有,ChannelPipeline 跳过 ChannelHandler 并继续下一个合适的方向。**记住,一个处理程序可能同时实现ChannelInboundHandler 和 ChannelOutboundHandler 接口。**



### 修改 ChannelPipeline

ChannelHandler 可以实时修改 ChannelPipeline 的布局，通过添加、移除、替换其他 ChannelHandler（也可以从 ChannelPipeline 移除 ChannelHandler 自身）。这个 是 ChannelHandler 重要的功能之一。

Table 6.6 ChannelHandler methods for modifying a ChannelPipeline

| 名称                                  | 描述                                      |
| ----------------------------------- | --------------------------------------- |
| addFirst addBefore addAfter addLast | 添加 ChannelHandler 到 ChannelPipeline.    |
| Remove                              | 从 ChannelPipeline 移除 ChannelHandler.    |
| Replace                             | 在 ChannelPipeline 替换另外一个 ChannelHandler |

下面展示了操作

Listing 6.5 Modify the ChannelPipeline

```java
ChannelPipeline pipeline = null; // get reference to pipeline;
FirstHandler firstHandler = new FirstHandler();  //1.创建一个 FirstHandler 实例
pipeline.addLast("handler1", firstHandler); 	//2.添加该实例作为 "handler1" 到 ChannelPipeline
pipeline.addFirst("handler2", new SecondHandler()); //3.添加 SecondHandler 实例作为 "handler2" 到 ChannelPipeline 的第一个槽，这意味着它将替换之前已经存在的 "handler1"
pipeline.addLast("handler3", new ThirdHandler()); //4.添加 ThirdHandler 实例作为"handler3" 到 ChannelPipeline 的最后一个槽

pipeline.remove("handler3"); 	//5.通过名称移除 "handler3"
pipeline.remove(firstHandler);   //6.通过引用移除 FirstHandler (因为只有一个，所以可以不用关联名字 "handler1"）

//7.将作为"handler2"的 SecondHandler 实例替换为作为 "handler4"的 FourthHandler
pipeline.replace("handler2", "handler4", new ForthHandler()); 
```

以后我们将看到,这种轻松添加、移除和替换 ChannelHandler 能力， 适合非常灵活的实现逻辑。



**ChannelHandler 的执行和阻塞** （通过 EventExecutorGroup 执行长任务）

- 通常 ChannelPipeline 中的每一个 ChannelHandler 都是通过它的 EventLoop（I/O 线程）来处理传递给它的事件的。所以至关重要的是不要阻塞这个线程，因为这会对整体的 I/O 处理产生负面的影响。
- 但有时可能需要与那些使用阻塞 API 的遗留代码进行交互。对于这种情况，ChannelPipeline 有一些接受一个 EventExecutorGroup 的 add()方法。如果一个事件被传递给一个自定义的 EventExecutorGroup，它将被包含在这个EventExecutorGroup 中的某个 EventExecutor 所处理，从而被从该Channel 本身的 EventLoop 中移除。对于这种用例，Netty 提供了一个叫 **DefaultEventExecutorGroup**的默认实现。




除了上述操作，其他访问 ChannelHandler 的方法如下：

Table 6.7 ChannelPipeline operations for retrieving ChannelHandlers

| 名称                 | 描述                                       |
| ------------------ | ---------------------------------------- |
| get(...)           | 通过类型或者名称返回ChannelHandler                 |
| context(...)       | 返回和ChannelHandler 绑定的ChannelHandlerContext |
| names() iterator() | 返回ChannelPipeline 中所有ChannelHandler 的名称  |



### 发送事件

ChannelPipeline API 有额外调用入站和出站操作的方法。

下表列出了**入站操作**,用于通知 ChannelPipeline 中 ChannelInboundHandlers 正在发生的事件

Table 6.8 Inbound operations on ChannelPipeline

| 名称                            | 描述                                       |
| ----------------------------- | ---------------------------------------- |
| fireChannelRegistered         | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelRegistered(ChannelHandlerContext)方法 |
| fireChannelUnregistered       | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelUnregistered(ChannelHandlerContext)方法 |
| fireChannelActive             | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelActive(ChannelHandlerContext)方法 |
| fireChannelInactive           | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelInactive(ChannelHandlerContext)方法 |
| fireExceptionCaught           | 调用ChannelPipeline 中下一个ChannelInboundHandler 的exceptionCaught(ChannelHandlerContext, Throwable)方法 |
| fireUserEventTriggered        | 调用ChannelPipeline 中下一个ChannelInboundHandler 的userEventTriggered(ChannelHandlerContext, Object)方法 |
| fireChannelRead               | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelRead(ChannelHandlerContext, Object msg)方法 |
| fireChannelReadComplete       | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelReadComplete(ChannelHandlerContext)方法 |
| fireChannelWritabilityChanged | 调用ChannelPipeline 中下一个ChannelInboundHandler 的channelWritabilityChanged(ChannelHandlerContext)方法 |



在**出站方面**,处理一个事件将导致底层套接字的一些行动。下表列出了ChannelPipeline API 出站的操作。

Table 6.9 Outbound operations on ChannelPipeline

| 名称            | 描述                                       |
| ------------- | ---------------------------------------- |
| bind          | 将Channel 绑定到一个本地地址，这将调用ChannelPipeline 中的下一个ChannelOutboundHandler的bind(ChannelHandlerContext, SocketAddress, ChannelPromise)方法 |
| connect       | 将Channel 连接到一个远程地址，这将调用ChannelPipeline 中的下一个ChannelOutboundHandler 的connect(ChannelHandlerContext, Socket-Address, ChannelPromise)方法 |
| disconnect    | disconnect 将Channel 断开连接。这将调用ChannelPipeline 中的下一个ChannelOutbound-Handler 的disconnect(ChannelHandlerContext, Channel Promise)方法close 将Channel 关闭。这将调用ChannelPipeline 中的下一个ChannelOutbound-Handler 的close(ChannelHandlerContext, ChannelPromise)方法 |
| close         | Close the Channel. This will call close(ChannelHandlerContext,ChannelPromise) on the next ChannelOutboundHandler in the ChannelPipeline. |
| deregister    | deregister 将Channel 从它先前所分配的EventExecutor（即EventLoop）中注销。这将调用ChannelPipeline 中的下一个ChannelOutboundHandler 的deregister(ChannelHandlerContext, ChannelPromise)方法 |
| flush         | 冲刷Channel所有挂起的写入。这将调用ChannelPipeline 中的下一个Channel- |
| write         | 将消息写入Channel。这将调用ChannelPipeline 中的下一个ChannelOutboundHandler的write(ChannelHandlerContext, Object msg, Channel-Promise)方法。注意：这并不会将消息写入底层的Socket，而只会将它放入队列中。要将它写入Socket，需要调用flush()或者writeAndFlush()方法 |
| writeAndFlush | 这是一个先调用write()方法再接着调用flush()方法的便利方法      |
| read          | 请求从Channel 中读取更多的数据。这将调用ChannelPipeline 中的下一个 |



总结下：

- 一个 ChannelPipeline 是用来保存关联到一个 Channel 的ChannelHandler
- 可以修改 ChannelPipeline 通过动态添加和删除 ChannelHandler
- ChannelPipeline 有着丰富的API调用动作来回应入站和出站事件。



## ChannelHandlerContext

- 接口 ChannelHandlerContext 代表 ChannelHandler 和ChannelPipeline 之间的关联,并在 ChannelHandler 添加到 ChannelPipeline 时创建一个实例。ChannelHandlerContext 的主要功能是**管理通过同一个 ChannelPipeline 关联的 ChannelHandler 之间的交互**。
- ChannelHandlerContext 有许多方法,其中一些也出现在 Channel 和ChannelPipeline 本身。然而,如果您通过Channel 或ChannelPipeline 的实例来调用这些方法，他们就会在整个 pipeline中传播 。相比之下,一样的 的方法在 ChannelHandlerContext的实例上调用， 就只会从当前的 ChannelHandler 开始并传播到相关管道中的**下一个有处理事件能力的 ChannelHandler** 。



ChannelHandlerContext API 总结如下：

Table 6.10 ChannelHandlerContext API

| 名称                            | 描述                                       |
| ----------------------------- | ---------------------------------------- |
| alloc                         | 返回和这个实例相关联的Channel 所配置的ByteBufAllocator  |
| bind                          | 绑定到给定的SocketAddress，并返回ChannelFuture     |
| channel                       | 返回绑定到这个实例的Channel                        |
| close                         | 关闭Channel，并返回ChannelFuture               |
| connect                       | 连接给定的SocketAddress，并返回ChannelFuture      |
| deregister                    | 从之前分配的EventExecutor 注销，并返回ChannelFuture  |
| disconnect                    | 从远程节点断开，并返回ChannelFuture                 |
| executor                      | 返回调度事件的EventExecutor                     |
| fireChannelActive             | 触发对下一个ChannelInboundHandler 上的channelActive()方法（已连接）的调用 |
| fireChannelInactive           | 触发对下一个ChannelInboundHandler 上的channelInactive()方法（已关闭）的调用 |
| fireChannelRead               | 触发对下一个ChannelInboundHandler 上的hannelRead()方法（已接收的消息）的调用 |
| fireChannelReadComplete       | 触发对下一个ChannelInboundHandler 上的channelReadComplete()方法的调用 |
| fireChannelRegistered         | 触发对下一个ChannelInboundHandler 上的fireChannelRegistered()方法的调用 |
| fireChannelUnregistered       | 触发对下一个ChannelInboundHandler 上的fireChannelUnregistered()方法的调用 |
| fireChannelWritabilityChanged | 触发对下一个ChannelInboundHandler 上的fireChannelWritabilityChanged()方法的调用 |
| fireExceptionCaught           | 触发对下一个ChannelInboundHandler 上的fireExceptionCaught(Throwable)方法的调用 |
| fireUserEventTriggered        | 触发对下一个ChannelInboundHandler 上的fireUserEventTriggered(Object evt)方法的调用 |
| handler                       | 返回绑定到这个实例的ChannelHandler                 |
| isRemoved                     | 如果所关联的ChannelHandler 已经被从ChannelPipeline中移除则返回true |
| name                          | 返回这个实例的唯一名称                              |
| pipeline                      | 返回这个实例所关联的ChannelPipeline                |
| read                          | 将数据从Channel读取到第一个入站缓冲区；如果读取成功则触发①一个channelRead事件，并（在最后一个消息被读取完成后）通知ChannelInboundHandler 的channelReadComplete(ChannelHandlerContext)方法 |
| write                         | 通过这个实例写入消息并经过ChannelPipeline             |
| writeAndFlush                 | 通过这个实例写入并冲刷消息并经过ChannelPipeline          |

其他注意注意事项：

- ChannelHandlerContext 与 ChannelHandler 的**关联从不改变**，所以缓存它的引用是安全的。
- 正如我们前面指出的,ChannelHandlerContext 所包含的**事件流比其他类中同样的方法都要短**，利用这一点可以尽可能高地提高性能。



### 使用 ChannelHandler

本节，我们将说明 ChannelHandlerContext的用法 ，以及ChannelHandlerContext, Channel 和 ChannelPipeline 这些类中方法的不同表现。下图展示了 ChannelPipeline, Channel, ChannelHandler 和 ChannelHandlerContext 的关系


![Figure_6.3](./channelHandler/Figure_6.3.jpg)

Figure 6.3 Channel, ChannelPipeline, ChannelHandler and ChannelHandlerContext

1. Channel 绑定到 ChannelPipeline
2. ChannelPipeline 绑定到 包含 ChannelHandler 的 Channel
3. ChannelHandler
4. 当添加 ChannelHandler 到 ChannelPipeline 时，ChannelHandlerContext 被创建



下面展示了， 从 ChannelHandlerContext 获取到 Channel 的引用，通过调用 Channel 上的 write() 方法来触发一个 写事件到通过管道的的流中

Listing 6.6 Accessing the Channel from a ChannelHandlerContext

```java
ChannelHandlerContext ctx = context;
Channel channel = ctx.channel();  //1.得到与 ChannelHandlerContext 关联的 Channel 的引用
channel.write(Unpooled.copiedBuffer("Netty in Action",CharsetUtil.UTF_8)); 	 //2.通过 Channel 写缓存
```

下面展示了 从 ChannelHandlerContext 获取到 ChannelPipeline 的相同示例

Listing 6.7 Accessing the ChannelPipeline from a ChannelHandlerContext

```java
ChannelHandlerContext ctx = context;
ChannelPipeline pipeline = ctx.pipeline(); //1.得到与 ChannelHandlerContext 关联的 ChannelPipeline 的引用
pipeline.write(Unpooled.copiedBuffer("Netty in Action", CharsetUtil.UTF_8));  //2.通过 ChannelPipeline 写缓冲区
```



流在两个清单6.6和6.7是一样的,如图6.4所示。重要的是要注意,虽然在 Channel 或者 ChannelPipeline 上调用write() 都会把事件在整个管道传播,但是在 ChannelHandler 级别上，从一个处理程序转到下一个却要通过在 ChannelHandlerContext 调用方法实现。

![Figure_6.4](./channelHandler/Figure_6.4.jpg)

Figure 6.4 Event propagation via the Channel or the ChannelPipeline

1. 事件传递给 ChannelPipeline 的第一个 ChannelHandler
2. ChannelHandler 通过关联的 ChannelHandlerContext 传递事件给 ChannelPipeline 中的 下一个
3. ChannelHandler 通过关联的 ChannelHandlerContext 传递事件给 ChannelPipeline 中的 下一个



为什么你可能会想从 ChannelPipeline 一个特定的点开始传播一个事件?

- 通过减少 ChannelHandler 不感兴趣的事件的传递，从而减少开销
- 排除掉特定的对此事件感兴趣的处理程序的处理

想要实现从一个特定的 ChannelHandler 开始处理，你必须引用与 此ChannelHandler的前一个ChannelHandler 关联的 ChannelHandlerContext 。这个ChannelHandlerContext 将会调用与自身关联的 ChannelHandler 的下一个ChannelHandler 。



下面展示了使用场景

Listing 6.8 Events via ChannelPipeline

```java
ChannelHandlerContext ctx = context;	//1.获得 ChannelHandlerContext 的引用
ctx.write(Unpooled.copiedBuffer("Netty in Action", CharsetUtil.UTF_8)); 
//2.write() 将会把缓冲区发送到下一个 ChannelHandler
```

如下所示,消息将会从下一个ChannelHandler开始流过 ChannelPipeline ,绕过所有在它**之前的ChannelHandler**。

![Figure_6.5](./channelHandler/Figure_6.5.jpg)

1. ChannelHandlerContext 方法调用
2. 事件发送到了下一个 ChannelHandler
3. 经过最后一个ChannelHandler后，事件从 ChannelPipeline 移除

Figure 6.5 Event flow for operations triggered via the ChannelHandlerContext

我们刚刚描述的用例是一种常见的情形,当我们想要调用某个特定的 ChannelHandler操作时，它尤其有用。



### ChannelHandler 和 ChannelHandlerContext 的高级用法

- 正如我们在清单6.6中看到的，通过调用ChannelHandlerContext的 pipeline() 方法，你可以得到一个封闭的 ChannelPipeline 引用。这使得可以在运行时操作 pipeline 的 ChannelHandler ，这一点可以被利用来实现一些复杂的需求,例如,添加一个 ChannelHandler 到 pipeline 来**支持动态协议改变**。
- 其他高级用例可以实现通过保持一个 ChannelHandlerContext 引用供以后使用,这可能发生在任何 ChannelHandler 方法,甚至来自不同的线程。清单6.9显示了此模式被用来触发一个事件。

Listing 6.9 ChannelHandlerContext usage

```java
public class WriteHandler extends ChannelHandlerAdapter {

    private ChannelHandlerContext ctx;

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        this.ctx = ctx;        //1.存储 ChannelHandlerContext 的引用供以后使用
    }

    public void send(String msg) {
        ctx.writeAndFlush(msg);  //2.使用之前存储的 ChannelHandlerContext 来发送消息
    }
}
```

因为 ChannelHandler 可以属于多个 ChannelPipeline ,它可以绑定多个 ChannelHandlerContext 实例。然而,ChannelHandler 用于这种用法必须添加 `@Sharable` 注解。否则,试图将它添加到多个 ChannelPipeline 将引发一个异常。此外,它必须既是线程安全的又能安全地使用多个同时的通道(比如,连接)。



清单6.10显示了此模式的正确实现。

Listing 6.10 A shareable ChannelHandler

```java
@ChannelHandler.Sharable            //1.添加 @Sharable 注解
public class SharableHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        System.out.println("channel read message " + msg);
        ctx.fireChannelRead(msg);  //2.日志方法调用， 并专递到下一个 ChannelHandler
    }
}
```

上面这个 ChannelHandler 实现符合所有包含在多个管道的要求;它通过`@Sharable` 注解，并不持有任何状态。而下面清单6.11中列出的情况则恰恰相反,它会造成问题。

Listing 6.11 Invalid usage of @Sharable

```java
@ChannelHandler.Sharable  //1.添加 @Sharable
public class NotSharableHandler extends ChannelInboundHandlerAdapter {
    private int count;

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        count++;  //2.count 字段递增

        System.out.println("inboundBufferUpdated(...) called the "+ count + " time");  
      	//3.日志方法调用， 并专递到下一个 ChannelHandler
        ctx.fireChannelRead(msg);
    }

}
```

这段代码的问题是它持有状态:一个实例变量保持了方法调用的计数。将这个类的一个实例添加到 ChannelPipeline 并发访问通道时很可能产生错误。(当然,这个简单的例子中可以通过在 channelRead() 上添加 synchronized 来纠正 )

总之,使用`@Sharable`的话，要确定 ChannelHandler 是线程安全的。



**为什么共享 ChannelHandler**

常见原因是要在多个 ChannelPipelines 上安装一个 ChannelHandler 以此来**实现跨多个渠道收集统计数据的目的**。

我们的讨论 ChannelHandlerContext 及与其他框架组件关系的 到此结束。接下来我们将解析 Channel 状态模型,准备仔细看看ChannelHandler 本身。



## 总结

- 本章带你深入窥探了一下 Netty 的数据处理组件: ChannelHandler。我们讨论了 ChannelHandler 之间是如何链接的以及它在像ChannelInboundHandler 和 ChannelOutboundHandler这样的化身中是如何与 ChannelPipeline 交互的。
- 下一章将集中在 Netty 的编解码器的抽象上,这种抽象使得编写一个协议编码器和解码器比使用原始 ChannelHandler 接口更容易。