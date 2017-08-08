---
categories: RabbitMQ
date: 2017-07-31 17:00
title: RabbitMQ —— RPC
---

在第二小节，我们学习了如何使用 Work Queues 将耗时的任务分发到多个 worker 处理。

但是，当我们需要在远程机计算机上运行一个函数，之前的例子就无法满足了。这种模式一般被称为 **远程方法调用**（*Remote Procedure Call* or *RPC*）。

在这一节，我们将使用 RabbitMQ 来构建一个 RPC 系统：一个客户端和一个可伸缩的 RPC 服务器。因为我们没有耗时的任务需要分布式计算，所以我们将创建一个虚构的 RPC Server 来返回 斐波那契数据列。



<!-- more -->

## 客户端接口（Client interface）

为了阐明 RPC 服务器 是如何工作的，我们将创建一个简单的客户端类。它暴露一个叫 **call** 的方法，用来发送 RPC 请求并阻塞直到收到应答。

```java
FibonacciRpcClient fibonacciRpc = new FibonacciRpcClient();
String result = fibonacciRpc.call("4");
System.out.println( "fib(4) is " + result);
```



> 使用 RPC 需要注意
>
> 虽然 RPC巴拉巴拉



## 回调队列（Callback queue）

一般情况下在 RabbitMQ 中进行 RPC 调用是很简单的。客户端发送请求消息（request ），服务器回复消息（response ）。为了能够收到 response ，我们需要在发送的 request 中包含一个回调（Callback ）队列地址。我们可以使用默认的队列（在 Java 客户端中的独占的）。

```java
callbackQueueName = channel.queueDeclare().getQueue();

BasicProperties props = new BasicProperties
                            .Builder()
                            .replyTo(callbackQueueName)	// 回调队列
                            .build();

channel.basicPublish("", "rpc_queue", props, message.getBytes());

// ... then code to read a response message from the callback_queue ...
```



> **消息的属性（Message properties）**
>
> AMQP 0-9-1 协议预定义了消息的 14个属性。大多数属性很少用到，除了以下几个：
>
> **deliveryMode** ：标识一个消息是持久化（persistent 使用值 2）或者瞬态（transient 任意其他值）。你也许还记得第二节使用到了这个属性。
>
> **contentType** ： 用于描述编码的 mime-type 。例如常用的 JSON 编码就是一个很好的例子，需要将值设置为 **application/json** 。
>
> **replyTo** : 一般用于命名回调队列。
>
> **correlationId** : 对于创建一个和 request 对应的 RPC response 很有帮助。



## Correlation Id

上面呈现的方法需要为每一个 RPC 请求创建一个回调队列。这种做法其实很低效，幸运的是有一种更好的方法 —— 每一个客户端只创建一个回调队列。

这又引发了一个新问题，队列收到了回复，但是却不知道这个回复属于哪个请求。这时候就需要用到 **CorrelationId** 属性了。我们将为每一个请求发送一个唯一的值，基于这点，就可以将回复和请求匹配起来。如果我们收到一个未知的 CorrelationId 值，我们可以安全的丢弃 —— 它不属于任何一个请求。

为什么要忽略回调队列中的未知的消息而不是返回一个错误呢？因为在很偶然的情况下，RPC server 在向客户端发送应答之后，但是在发送 ack 确认消息之前宕机了。这样 RPC server 回复后将再次处理请求。这即使为什么客户端需要优雅地处理重复的 response，RPC server 在理想状况下是幂等的。



## 总结

![python-six](./rpc/python-six.png)



**我们设计的 RPC 是这样工作的：**

- 客户端启动时，它将创建一个匿名独占的回调队列。
- 开始一个 RPC 请求，客户端发送一个消息包含两个属性：**replyTo** ，用于记录回调队列，以及**correlationId** 用于为每一个请求标记唯一值。
- 请求会发送到一个叫 **rpc_queue** 的队列。
- RPC worker 监听 **rpc_queue** 队列等待请求。当请求出现时，它完成工作并使用 **replyTo** 域记录的队列将结果发送回客户端。
- 客户端等待回调队列的数据。当消息出现时，它将检查 **correlationId** 属性，如果和请求的 **correlationId** 值匹配，就将响应 response 返回给应用程序。



## 合并代码

**RPCClient.java**

```java
package org.demo.rabbitMQDemo.rpc;

import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Envelope;

import java.io.IOException;
import java.util.UUID;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeoutException;

public class RPCClient {

  private Connection connection;
  private Channel channel;
  private String requestQueueName = "rpc_queue";
  private String replyQueueName;

  public RPCClient() throws IOException, TimeoutException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");

    connection = factory.newConnection();
    channel = connection.createChannel();

    replyQueueName = channel.queueDeclare().getQueue();
  }

  public String call(String message) throws IOException, InterruptedException {
    final String corrId = UUID.randomUUID().toString();

    AMQP.BasicProperties props = new AMQP.BasicProperties
            .Builder()
            .correlationId(corrId)
            .replyTo(replyQueueName)
            .build();

    channel.basicPublish("", requestQueueName, props, message.getBytes("UTF-8"));

    final BlockingQueue<String> response = new ArrayBlockingQueue<String>(1);

    channel.basicConsume(replyQueueName, true, new DefaultConsumer(channel) {
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        if (properties.getCorrelationId().equals(corrId)) {
          response.offer(new String(body, "UTF-8"));
        }
      }
    });

    return response.take();
  }

  public void close() throws IOException {
    connection.close();
  }

  public static void main(String[] argv) {
    RPCClient fibonacciRpc = null;
    String response = null;
    try {
      fibonacciRpc = new RPCClient();

      System.out.println(" [x] Requesting fib(30)");
      response = fibonacciRpc.call("30");
      System.out.println(" [.] Got '" + response + "'");
    }
    catch  (Exception e) {
      e.printStackTrace();
    }
    finally {
      if (fibonacciRpc!= null) {
        try {
          fibonacciRpc.close();
        }
        catch (IOException _ignore) {}
      }
    }
  }
}
```



RPCServer.java

```java
package org.demo.rabbitMQDemo.rpc;

import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Envelope;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class RPCServer {

  private static final String RPC_QUEUE_NAME = "rpc_queue";

  private static int fib(int n) {
    if (n ==0) return 0;
    if (n == 1) return 1;
    return fib(n-1) + fib(n-2);
  }

  public static void main(String[] argv) {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");

    Connection connection = null;
    try {
      connection      = factory.newConnection();
      final Channel channel = connection.createChannel();

      channel.queueDeclare(RPC_QUEUE_NAME, false, false, false, null);

      channel.basicQos(1);

      System.out.println(" [x] Awaiting RPC requests");

      Consumer consumer = new DefaultConsumer(channel) {
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
          AMQP.BasicProperties replyProps = new AMQP.BasicProperties
                  .Builder()
                  .correlationId(properties.getCorrelationId())
                  .build();

          String response = "";

          try {
            String message = new String(body,"UTF-8");
            int n = Integer.parseInt(message);

            System.out.println(" [.] fib(" + message + ")");
            response += fib(n);
          }
          catch (RuntimeException e){
            System.out.println(" [.] " + e.toString());
          }
          finally {
            channel.basicPublish( "", properties.getReplyTo(), replyProps, response.getBytes("UTF-8"));

            channel.basicAck(envelope.getDeliveryTag(), false);
          }
        }
      };

      channel.basicConsume(RPC_QUEUE_NAME, false, consumer);

      //loop to prevent reaching finally block
      while(true) {
        try {
          Thread.sleep(100);
        } catch (InterruptedException _ignore) {}
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
    finally {
      if (connection != null)
        try {
          connection.close();
        } catch (IOException _ignore) {}
    }
  }
}
```



**运行结果**

```shell
# CLient

 [x] Requesting fib(30)
 [.] Got '832040'
```

```shell
# Server

 [x] Awaiting RPC requests
 [.] fib(30)
```



客户端代码轻微涉及到：

- 客户端建立connection、channel 并为回复定义一个独占的 'callback' queue。
- 客户端订阅 ‘callback’ queue，这样就能接收到 RPC server 的响应。
- **call** 方法发送 RPC 请求。
- 首先生成一个唯一的 correlationId 并保存下来 —— DefaultConsumer  的 handleDelivery 实现将使用这个值来捕获适当的响应。
- 接着我们发布请求消息，包含两个属性：replyTo 和 correlationId。
- 此时，等待合适的响应到达。
- 因为消费者处理消息是在单独的线程中的，在响应到达之前，我们需要一种机制来挂起主线程。**BlockingQueue** 的使用是一种解决方法。我们创建一个 capacity 为1 的 **ArrayBlockingQueue**  来等待唯一的响应。
- **handleDelivery** 方法的工作很简单—— 检查每一个响应的 correlationId 是否是我们需要的，如果是，将响应添加进 **BlockingQueue**
- 同时主线程从 **BlockingQueue** 等待获取响应，最后将响应返回给用户。



**这种设计不是唯一的 RPC service 实现，但却有一些重要的知道建议：**

- 如果 RPC server 响应太慢，你可以通过运行多一个实例来扩大集群规模，运行第二个 RPCServer 。
- 客户端 RPC 请求只发送和接受一条消息。queueDeclare 的调用是不必要的。这样能每个 RPC 请求就只需要一次网络来回。

我们的代码任然过于简单，并没有考虑一些复杂（但很重要） 的问题：

- 如果没有 servers 在运行，那么客户端应该如何反应？
- 客户端需要为 RPC 设置超时吗？
- 如果服务器故障并抛出异常，它应该被转发到客户端吗？
- 在处理消息之前防止无效的消息（例如检查范围、类型等）。



