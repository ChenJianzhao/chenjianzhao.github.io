---
categories: RabbitMQ
date: 2017-07-26 00:00
title: RabbitMQ —— Rounting
---

在前面的小节，我们构建了一个简单的日志系统，让我们能够向许多接受者广播日志消息。

在这一小节，我们将增加一个特性，让接受者能够只订阅消息的一个子集。例如：我们能够只将重要的错误消息直接存储到日志文件中（写入到硬盘），而继续让所有的日志消息打印到控制台中。

<!-- more -->

## 绑定（Bindings）

前面我们已经创建过 Bindings，如：

```java
channel.queueBind(queueName, EXCHANGE_NAME, "");
```

绑定是 exchange 和 queue 之间的关联。上面的代码可以简单解释为：队列只对来自这个 exchange 的消息感兴趣。

Bindings 还可以添加一个额外的参数。为了避免和 `basic_publish` 中的参数 *routingKey* 混淆，我们将这里的参数称为 binding key。我们可以这样绑定：

```java
channel.queueBind(queueName, EXCHANGE_NAME, "black");
```

> 这个绑定的意义要视 exchange type 而定，如果是 fanout exchangs，会直接忽略这个key值



## Direct exchange

我们之前构建的日志系统会将所有的消息广播给所有的消费者。我们将拓展日志系统，让他能够通过日志额严重级别过滤消息。比如：我们将编写一个程序只将受到的 critical errors 日志写进磁盘，而不浪费磁盘空间记录 warning 或者 info 类型的日志消息。

之前使用的 fanout exchange 并不灵活——它只能做到盲目地广播。

接下来我们将使用 `direct exchange` 来代替，direct exchange 背后的路由算法其实很简单—— **消息将进入 binding key 和 消息的 routing key 完全匹配的 queue 中**。

![direct-exchange](routing/direct-exchange.png)



上面的例子中，我们看到 direct 类型的 exchange X 和两个队列绑定了，第一个队列使用 **orange** 的 banding key 和它绑定，第二个使用 **black** 和 **green** 两个 binding key 绑定。

在这样的设置下，使用 **orange** routing key 的消息将被路由到 队列 Q1，而使用 **black** 和 **green** routing key 的消息将被路由到队列 Q2，其他的消息将被丢弃。



## 多个绑定（Multiple bindings）

![direct-exchange-multiple](routing/direct-exchange-multiple.png)

多个队列使用相同的 binding key 绑定到 exchange 和完全合法的，上图的例子中，**direct** 类型的 exchange 将表现的和 **fanout** 类型一样，向所有匹配的的队列广播。使用 **black** routing key 的消息将被同时分派到 Q1 和 Q2。



## 发送日志（Emitting logs）

我们将在日志系统中使用这个模型。代替 fanout ，我们将会把消息发送到 **direct** 类型的 exchange，我们将提供日志严重等级（severity）作为 routing key。这样接收的程序就能够通过日志等级（severity）选择它要接收的日志。

首先创建 direct 类型的 exchange

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

接着发送消息

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

为了简化程序，我们假设日志严重级别（severity）为 'info', 'warning', 'error' 其中的一种。



## 订阅（Subscribing）

和前面接收消息的程序类似，只有一点不同，我们要为每个日志等级创建一个 binding ：

```java
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv){
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```



## 合并代码

![python-four](routing/python-four.png)



**EmitLogDirect.java**

```java
package org.demo.rabbitMQDemo.logQueues.direct;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

public class EmitLogDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {

    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
    
    /**
     *  日志等级  'info', 'warning', 'error' 
     */
    String severity = "info";  
    String message = "Test error";

    channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
    System.out.println(" [x] Sent '" + severity + "':'" + message + "'");

    channel.close();
    connection.close();
  }
}
```



**ReceiveLogsDirect.java**（日志将打印到屏幕上）

```Java
package org.demo.rabbitMQDemo.logQueues.direct;

import com.rabbitmq.client.*;

import java.io.IOException;

public class ReceiveLogsDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
    String queueName = channel.queueDeclare().getQueue();

    /** info 和 warning 级别的日志将打印到屏幕上 */
    String[] severitys = new String[]{"info", "warning"};
    for(String severity : severitys){
      channel.queueBind(queueName, EXCHANGE_NAME, severity);
    }
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    Consumer consumer = new DefaultConsumer(channel) {
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope,
                                 AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body, "UTF-8");
        System.out.println(" [x] Received '" + envelope.getRoutingKey() + "':'" + message + "'");
      }
    };
    channel.basicConsume(queueName, true, consumer);
  }
}
```

 

**ReceiveLogsDirectToDisk.java** （日子将写入到日志文件）

```java
package org.demo.rabbitMQDemo.logQueues.direct;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class ReceiveLogsDirectToDisk {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
    String queueName = channel.queueDeclare().getQueue();

	/** error 级别的日志将直接写入磁盘 */
    String[] severitys = new String[]{"error"};
    for(String severity : severitys){
      channel.queueBind(queueName, EXCHANGE_NAME, severity);
    }
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    Consumer consumer = new DefaultConsumer(channel) {
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope,
                                 AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body, "UTF-8");
        message = " [x] Received '" + envelope.getRoutingKey() + "':'" + message + "'" + "\n";
        FileOutputStream file = null;
        
        try{
        	file = new FileOutputStream("./direct.log",true);
        	file.write(message.getBytes("UTF-8"));
        }catch(FileNotFoundException e){
        	e.printStackTrace();
        }finally{
        	file.close();
        }
        
      }
    };
    channel.basicConsume(queueName, true, consumer);
  }
}
```



运行结果

```shell
# EmitLogDirect
 [x] Sent 'info':'Test error'
 [x] Sent 'error':'Test error'
```

```Shell
# ReceiveLogsDirect 
# see in console
 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'info':'Test error'
```

```shell
# ReceiveLogsDirectToDisk
# see in file "./direct.log"
 [x] Received 'error':'Test error'
```

