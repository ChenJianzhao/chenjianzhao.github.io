---
categories: Spring
date: 2017-03-06 00:36
status: public
title: 'kafka 简介与使用'
---

转载并整理自  [OrcHome](http://www.orchome.com/5)  作者：半兽人


## 一、首先来了解一下Kafka所使用的基本术语：
---
``Topic``
Kafka将消息种子(Feed)分门别类，每一类的消息称之为一个主题(Topic).
``Producer``
发布消息的对象称之为主题生产者(Kafka topic producer)
``Consumer``
订阅消息并处理发布的消息的种子的对象称之为主题消费者(consumers)
``Broker``
已发布的消息保存在一组服务器中，称之为Kafka集群。集群中的每一个服务器都是一个代理(Broker). 消费者可以订阅一个或多个主题（topic），并从Broker拉数据，从而消费这些已发布的消息。

## 二、话题和日志 (Topic和Log)
---
Topic是发布的消息的类别或者种子Feed名。对于每一个Topic，Kafka集群维护这一个分区的log，就像下图中的示例：

![](~/KmCudlf7DsaAVF0WAABMe0J0lv4158.png)

每一个分区都是一个顺序的、不可变的消息队列， 并且可以持续的添加。分区中的消息都被分了一个序列号，称之为偏移量(offset)，在每个分区中此偏移量都是唯一的。

Kafka集群保持所有的消息，直到它们过期， 无论消息是否被消费了。 实际上消费者所持有的仅有的元数据就是这个偏移量，也就是消费者在这个log中的位置。 这个偏移量由消费者控制：正常情况当消费者消费消息的时候，偏移量也线性的的增加。但是实际偏移量由消费者控制，消费者可以将偏移量重置为更老的一个偏移量，重新读取消息。 可以看到这种设计对消费者来说操作自如， 一个消费者的操作不会影响其它消费者对此log的处理。 再说说分区。Kafka中采用分区的设计有几个目的。一是可以处理更多的消息，不受单台服务器的限制。Topic拥有多个分区意味着它可以不受限的处理更多的数据。第二，分区可以作为并行处理的单元，稍后会谈到这一点。

## 三、分布式(Distribution)
---
Log的分区被分布到集群中的多个服务器上。每个服务器处理它分到的分区。 根据配置每个分区还可以复制到其它服务器作为备份容错。 每个分区有一个leader，零或多个follower。Leader处理此分区的所有的读写请求，而follower被动的复制数据。如果leader宕机，其它的一个follower会被推举为新的leader。 一台服务器可能同时是一个分区的leader，另一个分区的follower。 这样可以平衡负载，避免所有的请求都只让一台或者某几台服务器处理。

## 四、生产者(Producers)
---
生产者往某个Topic上发布消息。生产者也负责选择发布到Topic上的哪一个分区。最简单的方式从分区列表中轮流选择。也可以根据某种算法依照权重选择分区。开发者负责如何选择分区的算法。

## 五、消费者(Consumers)
---
通常来讲，消息模型可以分为两种， 队列和发布-订阅式。 队列的处理方式是 一组消费者从服务器读取消息，一条消息只有其中的一个消费者来处理。在发布-订阅模型中，消息被广播给所有的消费者，接收到消息的消费者都可以处理此消息。Kafka为这两种模型提供了单一的消费者抽象模型： 消费者组 （consumer group）。 消费者用一个消费者组名标记自己。 一个发布在Topic上消息被分发给此消费者组中的一个消费者。 假如所有的消费者都在一个组中，那么这就变成了queue模型。 假如所有的消费者都在不同的组中，那么就完全变成了发布-订阅模型。 更通用的， 我们可以创建一些消费者组作为逻辑上的订阅者。每个组包含数目不等的消费者， 一个组内多个消费者可以用来扩展性能和容错。正如下图所示：

## 六、Kafka的保证(Guarantees)
---
生产者发送到一个特定的Topic的分区上，消息将会按照它们发送的顺序依次加入，也就是说，如果一个消息M1和M2使用相同的producer发送，M1先发送，那么M1将比M2的offset低，并且优先的出现在日志中。
消费者收到的消息也是此顺序。
如果一个Topic配置了复制因子（replication facto）为N， 那么可以允许N-1服务器宕机而不丢失任何已经提交（committed）的消息。
有关这些保证的更多详细信息，请参见文档的设计部分。

## 七、安装并启动kafka
---
**Step 1: 下载代码**

下载0.10.0.0版本并且解压它。
```
tar -xzf kafka_2.11-0.10.0.0.tgz 
cd kafka_2.11-0.10.0.0
```

**Step 2: 启动服务**

运行kafka需要使用Zookeeper，所以你需要先启动Zookeeper，如果你没有Zookeeper，你可以使用kafka自带打包和配置好的Zookeeper。
```
bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```
现在启动kafka服务

```
bin/kafka-server-start.sh config/server.properties &
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

**Step 3: 创建一个主题(topic)**

创建一个名为“test”的Topic，只有一个分区和一个备份：
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
创建好之后，可以通过运行以下命令，查看已创建的topic信息：
```
bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```
或者，除了手工创建topic外，你也可以配置你的broker，当发布一个不存在的topic时自动创建topic。

**Step 4: 发送消息**

Kafka提供了一个命令行的工具，可以从输入文件或者命令行中读取消息并发送给Kafka集群。每一行是一条消息。
运行producer（生产者）,然后在控制台输入几条消息到服务器。

``` 
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test 
This is a message
This is another message
```
**Step 5: 消费消息**

Kafka也提供了一个消费消息的命令行工具，将存储的信息输出出来。
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
This is a message
This is another message
```
如果你有2台不同的终端上运行上述命令，那么当你在运行生产者时，消费者就能消费到生产者发送的消息。
所有的命令行工具有很多的选项，你可以查看文档来了解更多的功能。

**Step 6: 设置多个broker集群**

到目前，我们只是单一的运行一个broker,，没什么意思。对于Kafka,一个broker仅仅只是一个集群的大小, 所有让我们多设几个broker.
首先为每个broker创建一个配置文件:

```
cp config/server.properties config/server-1.properties 
cp config/server.properties config/server-2.properties
```
现在编辑这些新建的文件，设置以下属性：

```
config/server-1.properties: 
    broker.id=1 
    listeners=PLAINTEXT://:9093 
    log.dir=/tmp/kafka-logs-1

config/server-2.properties: 
    broker.id=2 
    listeners=PLAINTEXT://:9094 
    log.dir=/tmp/kafka-logs-2
```
broker.id是集群中每个节点的唯一且永久的名称，我们修改端口和日志分区是因为我们现在在同一台机器上运行，我们要防止broker在同一端口上注册和覆盖对方的数据。

我们已经运行了zookeeper和刚才的一个kafka节点，所有我们只需要在启动2个新的kafka节点。

```
bin/kafka-server-start.sh config/server-1.properties &
... 
> bin/kafka-server-start.sh config/server-2.properties &
...
```
现在，我们创建一个新topic，把备份设置为：3

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
好了，现在我们已经有了一个集群了，我们怎么知道每个集群在做什么呢？运行命令“describe topics”

```
 bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic    PartitionCount:1    ReplicationFactor:3    Configs:
Topic: my-replicated-topic    Partition: 0    Leader: 1    Replicas: 1,2,0    Isr: 1,2,0
```

这是一个解释输出，第一行是所有分区的摘要，每一个线提供一个分区信息，因为我们只有一个分区，所有只有一条线。

"leader"：该节点负责所有指定分区的读和写，每个节点的领导都是随机选择的。
"replicas":备份的节点，无论该节点是否是leader或者目前是否还活着，只是显示。
"isr"：备份节点的集合，也就是活着的节点集合。
我们运行这个命令，看看一开始我们创建的那个节点：

```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test    PartitionCount:1    ReplicationFactor:1    Configs:
Topic: test    Partition: 0    Leader: 0    Replicas: 0    Isr: 0
```
没有惊喜，刚才创建的topic（主题）没有Replicas，所以是0。

让我们来发布一些信息在新的topic上：

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
 ...
my test message 1
my test message 2
^C
```
现在，消费这些消息。

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic
 ...
my test message 1
my test message 2
^C
```
我们要测试集群的容错，kill掉leader，Broker1作为当前的leader，也就是kill掉Broker1。

```
ps | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home/bin/java... 
> kill -9 7564
```
备份节点之一成为新的leader，而broker1已经不在同步备份集合里了。

```
 bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic    PartitionCount:1    ReplicationFactor:3    Configs:
Topic: my-replicated-topic    Partition: 0    Leader: 2    Replicas: 1,2,0    Isr: 2,0
```
但是，消息仍然没丢：

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

**Step 7: 使用 Kafka Connect 来 导入/导出 数据**

从控制台写入和写回数据是一个方便的开始，但你可能想要从其他来源导入或导出数据到其他系统。对于大多数系统，可以使用kafka Connect，而不需要编写自定义集成代码。Kafka Connect是导入和导出数据的一个工具。它是一个可扩展的工具，运行连接器，实现与自定义的逻辑的外部系统交互。在这个快速入门里，我们将看到如何运行Kafka Connect用简单的连接器从文件导入数据到Kafka主题，再从Kafka主题导出数据到文件，首先，我们首先创建一些种子数据用来测试：

```
echo -e "foo\nbar" > test.txt
```
接下来，我们开始2个连接器运行在独立的模式，这意味着它们运行在一个单一的，本地的，专用的进程。我们提供3个配置文件作为参数。第一个始终是kafka Connect进程，如kafka broker连接和数据库序列化格式，剩下的配置文件每个指定的连接器来创建，这些文件包括一个独特的连接器名称，连接器类来实例化和任何其他配置要求的。

```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```
这是示例的配置文件，使用默认的本地集群配置并创建了2个连接器：第一个是导入连接器，从导入文件中读取并发布到Kafka主题，第二个是导出连接器，从kafka主题读取消息输出到外部文件，在启动过程中，你会看到一些日志消息，包括一些连接器实例化的说明。一旦kafka Connect进程已经开始，导入连接器应该读取从

test.txt
和写入到topic

connect-test
,导出连接器从主题

connect-test
读取消息写入到文件

test.sink.txt
. 我们可以通过验证输出文件的内容来验证数据数据已经全部导出：

cat test.sink.txt
 foo
 bar
注意，导入的数据也已经在Kafka主题

connect-test
里,所以我们可以使用该命令查看这个主题：

```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic connect-test --from-beginning
 {"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```
连接器继续处理数据，因此我们可以添加数据到文件并通过管道移动：
```
echo "Another line" >> test.txt
```
你应该会看到出现在消费者控台输出一行信息并导出到文件。


## 八、最后怎么样才算真正的学会kafka
---
kafka节点之间如何复制备份的？
kafka消息是否会丢失？为什么？
kafka最合理的配置是什么？
kafka的leader选举机制是什么？
kafka对硬件的配置有什么要求？
kafka的消息保证有几种方式？
...