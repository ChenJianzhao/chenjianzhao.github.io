---
categories: RabbitMQ
date: 2017-07-25 20:00
title: RabbitMQ —— Work Queues
---



## 准备工作

工作队列的主要设计思想是为了避免立即执行“资源密集型”的任务而必须等待其完成。取而代之，我们可以安排任务延迟执行。我们将一个任务封装成信息并发送到队列，在后台运行的“worker”进程将弹出任务并最终执行任务。如果你运行很多个“worker”，他们将共享这些任务。

<!-- more -->

![python-two](./worker/python-two.png)



发布任务的进程代码，NewTask.java

```java
package org.demo.rabbitMQDemo.workerQueues;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);

        int msgCount = 0;
        String message = "[" + msgCount + "] Hello World.";
        System.out.println("Sent '" + message + "'");

        channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
//        System.out.println(" [x] Sent '" + message + "'");

        channel.close();
        connection.close();
    }
}
```



执行任务的进程代码，Worker.java

```java
package org.demo.rabbitMQDemo.workerQueues;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class Worker {

	private final static String TASK_QUEUE_NAME = "task_queue";

	public static void main(String[] args) {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost("localhost");
		Connection connection = null;
		Channel channel = null;
		Consumer consumer = null;

		try {
			connection = factory.newConnection();
			channel= connection.createChannel();

			channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
			System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

			consumer = new DefaultConsumer(channel) {
				@Override
				public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
					String message = new String(body, "UTF-8");

					System.out.println("Received '" + message + "'");
					try {
						doWork(message);
					} catch(InterruptedException e){
						e.printStackTrace();
					}finally {
//						System.out.println(" [x] Done");
					}
				}
			};

			/**
			 *  autoAck 是否自动发送确认消息
			 *  消费者没有确认的情况下，RabbitMQ Server不会删除该消息，也不会再向该消费者再次推送同一消息
			 */
			boolean autoAck = true;
			channel.basicConsume(TASK_QUEUE_NAME, autoAck, consumer);

		} catch (Exception e1) {
			e1.printStackTrace();
		}
	}

	private static void doWork(String task) throws InterruptedException {
	    for (char ch: task.toCharArray()) {
	        if (ch == '.') Thread.sleep(1000);
	    }
	}	
}
```



## 轮询分派（Round-robin dispatching）

使用“任务队列”的一个优点是，它有轻松的让任务并行执行。如果我们建立的阻塞的任务，只需要增加多几个Worker就能轻松实现均衡。



首先，我们先运行2个Worker实例，他们都将从队列获取任务信息。

```shell
 # Run Worker1
 [*] Waiting for messages. To exit press CTRL+C
```

```shell
 # Run Worker2
 [*] Waiting for messages. To exit press CTRL+C
```



然后，通过 NewTask 发布4条消息

```shell
# Run NewTask 4 Times

Sent '[0] Hello World.'
Sent '[1] Hello World.'
Sent '[2] Hello World.'
Sent '[3] Hello World.'
```



可以看到2个 Worker 控制台的输出：

```shell
 # Run Worker1
 [*] Waiting for messages. To exit press CTRL+C
Received '[0] Hello World.'
Received '[2] Hello World.'
```

```shell
 # Run Worker2
 [*] Waiting for messages. To exit press CTRL+C
Received '[1] Hello World.'
Received '[3] Hello World.'
```

默认情况下，RabbitMQ 会按顺序依次将每条消息发送给下一个消费者，每个消费者将平均地收到相同数量的消息。这种方式叫做`轮询`。



## 消息确认（Message acknowledgment）

假设一旦 RabbitMQ 向消费者提交了一个消息，这条消息将立刻被从内存中删除。这样的情况下，如果Worker进程被杀死，我们将丢失worker正在处理的消息，我们也将丢失分配到这个特定的worker但未被处理的所有消息。

未了不让任务丢失，我们需要在worker死掉之后，能够将任务分配给另外的worker。

为了保证信息不丢失，RabbitMQ 支持 消息`确认` （*acknowledgments*），消费者将向回复一个 ack 告诉 RabbitMQ 某个特定的消息已经被接受，可以删除。

如果一个消费者死了（channel 关闭，connection关闭或者TCP连接丢失）而没有发送 ack，RabbitMQ 将认为消息没有被处理完全并会将其重新加入队列，如果同时存在其他在线的消费者，消息将被快速地分配给另外的消费者。这样就能在worker偶尔宕机的情况下确保消息不丢失。

消息 `acknowledgments` 默认是打开的。我们可以显示地通过  `autoAck=false` 关闭自动确认，在worker中适当的位置手动发送确认。

修改 worker 的代码，在 finally 中加入发送 ack 的代码，并关闭消费者的自动 ack 确认

```java
channel.basicQos(1); // accept only one unack-ed message at a time (see below)

final Consumer consumer = new DefaultConsumer(channel) {
  @Override
  public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
    String message = new String(body, "UTF-8");

    System.out.println(" [x] Received '" + message + "'");
    try {
      doWork(message);
    } finally {
      System.out.println(" [x] Done");
      channel.basicAck(envelope.getDeliveryTag(), false);
    }
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, consumer);
```



## 消息持久化 （Message durability）

当 RabbitMQ 退出或崩溃，它将丢弃所有的队列和信息。你需要两个条件来保证消息不会丢失：**将队列和消息都标识为持久的（durable）**。

- 声明队列持久化（durable）

```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

> **注：**若对已经存在的被声明为“非持久化” 的队列，再次声明为持久化，RabbitMQ 是不允许修改已存在的队列的参数的，并且会返回错误。



- 声明消息持久化

设置消息属性 `MessageProperties` (which implements BasicProperties) 的值为 `PERSISTENT_TEXT_PLAIN`

```java
import com.rabbitmq.client.MessageProperties;

channel.basicPublish("", "task_queue", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```

> **注：** 标识消息持久化不完全保证消息不会丢失，在 RabbitMQ 接受到消息但还未持久化之间任然存在很短的时间间隔。同时，RabbitMQ 不会进行对每个信息做 二级同步——消息可能只保存在缓存中而没有被写到硬盘中。这个持久化保证并不强健。
>
> 如果需要强持久化保证，可以使用  [publisher confirms](https://www.rabbitmq.com/confirms.html).



## 公平调度 （Fair dispatch）

RabbitMQ 默认会平均地分配任务给每个消费者，但这样可能导致有些消费者一直空闲而另外一些却处于一直繁忙的状态。

这种现象的原因是，RabbitMQ 在消息进入队列的时候就派发消息，而没有查看没有回复消息确认 ack 的消费者，它只是简单的将第 n 个消息派发给第 n 个消费者。

为了防止这种现象，我们可以为 `basicQos` 方法设置  `prefetchCount = 1`。这将告诉 RabbitMQ  在同一时间不为worker 派发超过一个消息，换言之就是，在 worker 处理完任务并回复消息确认上一个消息之前，之前不再派发新的消息，它将派发给下一个不繁忙的 workter。

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```



## 合并代码

**NewTask.java**

```java
package org.demo.rabbitMQDemo.workerQueues;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

        int msgCount = 0;
        String message = "[" + msgCount + "] Hello World.";
        System.out.println(" ["+ msgCount +"] Sent '" + message + "'");

        channel.basicPublish("", TASK_QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + message + "'");

        channel.close();
        connection.close();
    }
}
```



**Worker.java**

```java
package org.demo.rabbitMQDemo.workerQueues;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class Worker {

	private final static String TASK_QUEUE_NAME = "task_queue";

	public static void main(String[] args) {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost("localhost");
		Connection connection = null;
		Channel channel = null;
		Consumer consumer = null;

		try {
			connection = factory.newConnection();
			channel= connection.createChannel();

         	 /** 
		     * 限制每个消费者同一时间只能消费一个消息 
		     * （即没有发送确认ack，不会向该消费者推送其他消息）
		     */	
			int prefetchCount = 1;
			channel.basicQos(prefetchCount);

			channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
			System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

			consumer = new DefaultConsumer(channel) {
				@Override
				public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
					String message = new String(body, "UTF-8");

					System.out.println("Received '" + message + "'");
					try {
						doWork(message);
					} catch(InterruptedException e){
						e.printStackTrace();
					}finally {
						System.out.println(" [x] Done");
						/**
						 * 向服务器发送确认消息
						 */
						channel.basicAck(envelope.getDeliveryTag(), false);
					}
				}
			};

			/**
			 *  autoAck 是否自动发送确认消息
			 *  消费者没有确认的情况下，RabbitMQ Server不会删除该消息，也不会再向该消费者再次推送同一消息
			 */
			boolean autoAck = false;
			channel.basicConsume(TASK_QUEUE_NAME, autoAck, consumer);

		} catch (Exception e1) {
			e1.printStackTrace();
		}
	}

	private static void doWork(String task) throws InterruptedException {
	    for (char ch: task.toCharArray()) {
	        if (ch == '.') Thread.sleep(1000);
	    }
	}	
}
```