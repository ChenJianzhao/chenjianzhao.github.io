---
categories: RabbitMQ
date: 2017-07-27 17:00
title: RabbitMQ —— Topic
---



在前面的小结，我们使用了 **direct** exchange 代替盲目广播的 **fanout** exchagne 使得接受者能够选择需要接收的日志消息。但 **direct** exchange 还是存在局限性——**它无法根据多标准路由**。

日志系统中，我们可能不止需要根据**日志级别**订阅消息，同时也需要基于 **”日志发送来源“** 来过滤。你可能从  [syslog](http://en.wikipedia.org/wiki/Syslog) unix tool 得知可以通过日志的 **严重级别** severity (info/warn/crit...) 和 **来源设备** facility (auth/cron/kern...) 来路由日志。

这会基于我们更多的灵活性 —— 我们也许只想监听来自 'cron' 的 critical errors 和 所有来自 'kern' 的日志。



<!-- more -->

## Topic exchange

发送到 **topic** exchange 的消息能跟随一个任意的 **routing_key**  —— 它必须是**由许多点分隔的单词列表**，单词可以是任意的词，但通常是指定了消息的某些特性。

如下是几个合法的 routing key："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit"。

你可以在 routing key 中定义任意数量的单词，但上限为 255 字节（bytes）。

**binding key** 也必须是同样的格式，**topic** exchange 背后的逻辑和 **direct** 是类似的 —— **发送到指定 routing key 的消息将被分发到所有绑定了相匹配的 binding key 的队列**。 但是 binding keys 有两个重要特殊例子：

- *（star）星号能完全代替 1 个单词
- \#（hash） 井号能代替 0 个或多个单词。



如以下例子：

![python-five](./topic/python-five.png)

在这个例子中，我们将发送的所有消息都是描述动物的。发送的消息将跟随由3个单词（2个分隔号）组成的 routing key。第一个单词描述速度（speed），第二个颜色（colour），第三个物种（species）： `"<speed>.<colour>.<species>"`

接着创建 binding： Q1 和 ` *.orange.* ` 绑定，Q2 和 `*.*.rabbit` 以及 `lazy.#` 绑定。

这些绑定总结为：

- Q1 对所有 orange 的动物感兴趣
- Q2 监听所有关于 rabbit 和 lazy 的动物

> **例子：**
>
> "quick.**orange**.rabbit" 和  "**lazy**.orange.elephant"  的消息将被分发给两个队列，
>
> "quick.**orange**.fox" 将只被分发个第一个队列，"**lazy**.brown.fox" 将只被分发给第二个队列，
>
> "**lazy**.pink.**rabbit**" 将**只分发给第二个队列一次**，即使它匹配了两个 binding。"quick.brown.fox" 不匹配任何 binding，消息将被丢弃。
>
> 如果我们不按约定而发送1个单词或者4个单词的消息，如："orange" or "quick.orange.male.rabbit" ， 这些消息不匹配任何一个 binding，将被丢弃。但是"**lazy**.orange.male.rabbit" 虽然有四个单词，但是它将匹配最后一个 binding `lazy.#` 并分发给第二个队列。



>**Topic exchange**
>
>Topic exchange 非常强大，它能表现得像其他的 exchange 一样。
>
>当队列绑定了 binding key “#”（hash） —— 它将收到所有的消息，不管 routing key 是怎样的 —— 这表现得像 fanout exchange。
>
>当这些特殊字符 “*”（star）和 “#”（hash）不在 binding 中使用，topic exchange 将表现得像 **direct** exchange



## 合并代码

我们将在日志系统中使用 **topic** exchange ，假定日志的 routing key 有两个单词：`<facility>.<severity>`

**EmitLogTopic.java**

```java
public class EmitLogTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) {
    Connection connection = null;
    Channel channel = null;
    try {
      ConnectionFactory factory = new ConnectionFactory();
      factory.setHost("localhost");

      connection = factory.newConnection();
      channel = connection.createChannel();

      channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
      
      /**
       *  both severity (info/warn/critical...) and facility (auth/cron/kern...).
       */
      String routingKey = "auth.info"; 
      String message = "A critical kernel error";

      channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
      System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");

    }
    catch  (Exception e) {
      e.printStackTrace();
    }
    finally {
      if (connection != null) {
        try {
          connection.close();
        }
        catch (Exception ignore) {}
      }
    }
  }
}
```

**ReceiveLogsTopic.java**

```java
package org.demo.rabbitMQDemo.logQueues.topic;

import com.rabbitmq.client.*;

import java.io.IOException;

public class ReceiveLogsTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
    String queueName = channel.queueDeclare().getQueue();

    /** 
      * 所有来源、所有等级的日志都需要打印到屏幕上 
      */
    String[] topics = new String[]{"#"};
    for (String bindingKey : topics) {
      channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
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

**ReceiveLogsTopicToDisk.java**

```java
package org.demo.rabbitMQDemo.logQueues.topic;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class ReceiveLogsTopicToDisk {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
    String queueName = channel.queueDeclare().getQueue();

    /**
     * both severity (info/warn/critical...) and facility (auth/cron/kern...).
     * 所有 from kern and critical severity 的日志会保存到日志文件中
     */
    String[] topics = new String[]{"kern.*","*.critical"};
    for (String bindingKey : topics) {
      channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
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
        	file = new FileOutputStream("./topic.log",true);
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



```shell
# EmitLogTopic

 [x] Sent 'auth.info':'A critical kernel error'
 [x] Sent 'auth.warn':'A critical kernel error'
 [x] Sent 'auth.critical':'A critical kernel error'
 [x] Sent 'cron.info':'A critical kernel error'
 [x] Sent 'cron.warn':'A critical kernel error'
 [x] Sent 'cron.critical':'A critical kernel error'
 [x] Sent 'kern.info':'A critical kernel error'
 [x] Sent 'kern.warn':'A critical kernel error'
 [x] Sent 'kern.critical':'A critical kernel error'
```

```shell
# ReceiveLogsTopic
# see in console

 [*] Waiting for messages. To exit press CTRL+C
 [x] Received 'auth.info':'A critical kernel error'
 [x] Received 'auth.warn':'A critical kernel error'
 [x] Received 'auth.critical':'A critical kernel error'
 [x] Received 'cron.info':'A critical kernel error'
 [x] Received 'cron.warn':'A critical kernel error'
 [x] Received 'cron.critical':'A critical kernel error'
 [x] Received 'kern.info':'A critical kernel error'
 [x] Received 'kern.warn':'A critical kernel error'
 [x] Received 'kern.critical':'A critical kernel error'
```

```shell
# ReceiveLogsTopicToDisk
# see in file "topic.log"

 [x] Received 'auth.critical':'A critical kernel error'
 [x] Received 'cron.critical':'A critical kernel error'
 [x] Received 'kern.info':'A critical kernel error'
 [x] Received 'kern.warn':'A critical kernel error'
 [x] Received 'kern.critical':'A critical kernel error'

```

可以看出，屏幕上打印出来所有发送出去的日志消息，而日志文件只记录了等级为 “critical” 和所有来自 “kern” 的日志。