在 work queue 一节 我们创建了work queue，其背后的构思是每个任务都分发给一个确定的 worker 。这一节，我们将会把消息交付给多个消费者，这种模式叫做“发布/订阅” （publish/subscribe）模式。

我们将建立一个简单的日志系统，它由两个程序组成，一个负责发送日志信息，另外一个负责接收并打印日志。

在日志系统中，每一个运行的接受者程序都将收到消息。通过这种方法，我们可以运行一个接受者直接将日志写入硬盘，同事运行另外一个接受者将日志打印到屏幕上。

本质上，发布日志消息将广播给所有的接受者。



## 交换(机)（Exchanges）

在前面的小节，我们向队列发送和接受消息，接下来要介绍 Rabbit 完整的消息模型。

RabbitMQ 消息模型的核心思想是：从不直接向队列发送任何消息。实际上，生产者经常是完全不知道消息会被交付给哪一个队列的。

相应的，生产者只能发送消息给 `exchange`。exchange 很简单，一方面它从生产者接受消息，另一方面将消息推送给队列。exchange 必须准确地知道如何处理它接受到的消息。它应该被追加到特定的队列？还是广播给所有的队列？还是应该被废弃。

`exchange type` 定义了如下规则：

![exchange](./publish-subscribe/exchange)

有如下几个 exchange type：

`direct`, `topic`, `headers` and `fanout`。

以下集中介绍最后一个 fanout（扇出/广播）。

先创建一个这种类型的 exchange 命名为“logs”

```java
channel.exchangeDeclare("logs", "fanout");
```

fanout exchange 很简单，就是将它接受到的所有消息广播给它知道的所有的队列。