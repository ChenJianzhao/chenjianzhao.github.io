## 概述

Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后能进行自懂切换。

它的主要功能有以下几点

- 不时地监控redis是否按照预期良好地运行;

- 如果发现某个redis节点运行出现状况，能够通知另外一个进程(例如它的客户端);

- 能够进行自动切换。当一个master节点不可用时，能够选举出master的多个slave(如果有超过一个slave的话)中的一个来作为新的master,其它的slave节点会将它所追随的master的地址改为被提升为master的slave的新地址。

  ​

## Sentinel支持集群

很显然，只使用单个sentinel进程来监控redis集群是不可靠的，当sentinel进程宕掉后(sentinel本身也有单点问题，single-point-of-failure)整个集群系统将无法按照预期的方式运行。所以有必要将sentinel集群，这样有几个好处：

- 即使有一些sentinel进程宕掉了，依然可以进行redis集群的主备切换；
- 如果只有一个sentinel进程，如果这个进程运行出错，或者是网络堵塞，那么将无法实现redis集群的主备切换（单点问题）;
- 如果有多个sentinel，redis的客户端可以随意地连接任意一个sentinel来获得关于redis集群中的信息。



## 运行Sentinel

运行sentinel有两种方式：

- 第一种

  ```
  redis-sentinel ./sentinel1/sentinel.conf
  ```

- 第二种

  ```
  redis-server ./sentinel1/sentinel.conf --sentinel
  ```

  以上两种方式，都必须指定一个sentinel的配置文件sentinel.conf，如果不指定，将无法启动sentinel。sentinel默认监听26379端口，所以运行前必须确定该端口没有被别的进程占用。



## Sentinel的配置

Redis源码包中包含了一个sentinel.conf文件作为sentinel的配置文件，配置文件自带了关于各个配置项的解释。典型的配置项如下所示：

```shell
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

## 疑问：为什么可以配置多个Master？？ 不同的集群？？
sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

上面的配置项配置了两个名字分别为mymaster和resque的master，配置文件只需要配置master的信息就好啦，不用配置slave的信息，因为slave能够被自动检测到(master节点会有关于slave的消息)。需要注意的是，配置文件在sentinel运行期间是会被动态修改的，例如当发生主备切换时候，配置文件中的master会被修改为另外一个slave。这样，之后sentinel如果重启时，就可以根据这个配置来恢复其之前所监控的redis集群的状态。



**接下来我们将一行一行地解释上面的配置项**：

```
sentinel monitor mymaster 127.0.0.1 6379 2

```

这一行代表sentinel监控的master的名字叫做**mymaster**,地址为**127.0.0.1:6379**，行尾最后的一个**2**代表什么意思呢？我们知道，网络是不可靠的，有时候一个sentinel会因为网络堵塞而误以为一个master redis已经死掉了，当sentinel集群式，解决这个问题的方法就变得很简单，只需要多个sentinel互相沟通来确认某个master是否真的死了，这个**2**代表，当集群中有2个sentinel认为master死了时，才能真正认为该master已经不可用了。（sentinel集群中各个sentinel也有互相通信，通过gossip协议）。



除了第一行配置，我们发现剩下的配置都有一个统一的格式:

```
sentinel <option_name> <master_name> <option_value>

```

接下来我们根据上面格式中的**option_name**一个一个来解释这些配置项：

- **down-after-milliseconds**
  sentinel会向master发送心跳**PING**来确认master是否存活，如果master在**“一定时间范围”**内不回应**PONG** 或者是回复了一个错误消息，那么这个sentinel会**主观地**(单方面地)认为这个master已经不可用了(subjectively down, 也简称为SDOWN)。而这个down-after-milliseconds就是用来指定这个**“一定时间范围”**的，单位是毫秒。

> 不过需要注意的是，这个时候sentinel并不会马上进行failover主备切换，这个sentinel还需要参考sentinel集群中其他sentinel的意见，如果超过某个数量的sentinel也**主观地**认为该master死了，那么这个master就会被**客观地**(注意哦，这次不是主观，是客观，与刚才的subjectively down相对，这次是objectively down，简称为ODOWN)认为已经死了。需要一起做出决定的sentinel数量在上一条配置中进行配置。



- **parallel-syncs**
  在发生failover主备切换时，这个选项指定了最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave处于不能处理命令请求的状态。

其他配置项在sentinel.conf中都有很详细的解释。
所有的配置都可以在运行时用命令`SENTINEL SET command`动态修改。



- **failover-timeout**  见以下小节

## Sentinel的“仲裁会”

- 前面我们谈到，当一个master被sentinel集群监控时，需要为它指定一个参数，这个参数指定了当需要判决master为不可用，并且进行failover时，所需要的sentinel数量，本文中我们暂时称这个参数为**票数**
- 不过，当failover主备切换真正被触发后，failover并不会马上进行，还需要sentinel中的**大多数**sentinel授权后才可以进行failover。
- 当ODOWN时，failover被触发。failover一旦被触发，尝试去进行failover的sentinel会去获得“大多数”sentinel的授权（如果票数比大多数还要大的时候，则询问更多的sentinel)
- 这个区别看起来很微妙，但是很容易理解和使用。例如，集群中有5个sentinel，票数被设置为2，当2个sentinel认为一个master已经不可用了以后，将会触发failover，但是，进行failover的那个sentinel必须先获得至少3个sentinel的授权才可以实行failover。
- 如果票数被设置为5，要达到ODOWN状态，必须所有5个sentinel都主观认为master为不可用，要进行failover，那么得获得所有5个sentinel的授权。



## 配置版本号

（原文描述有错误？：勘误: sentinel集群都遵守一个规则：如果sentinel 推荐sentinel B去执行failover，~~B会等待一段时间后~~，自行再次去对同一个master执行failover，这个等待的时间是通过`failover-timeout`配置项去配置的。）

- 为什么要先获得**大多数**sentinel的认可时才能真正去执行failover呢？
- 当一个sentinel被授权后，它将会获得宕掉的master的一份最新配置版本号，当failover执行结束以后，这个版本号将会被用于最新的配置。因为**大多数**sentinel都已经知道该版本号已经被要执行failover的sentinel拿走了，所以其他的sentinel都不能再去使用这个版本号。这意味着，每次failover都会附带有一个独一无二的版本号。我们将会看到这样做的重要性。
- 而且，sentinel集群都遵守一个规则：如果sentinel A推荐sentinel B去执行failover，A会等待一段时间后，自行再次去对同一个master执行failover，这个等待的时间是通过`failover-timeout`配置项去配置的。从这个规则可以看出，sentinel集群中的sentinel不会再同一时刻并发去failover同一个master，第一个进行failover的sentinel如果失败了，另外一个将会在一定时间内进行重新进行failover，以此类推。

**redis sentinel保证了活跃性**：如果**大多数**sentinel能够互相通信，最终将会有一个被授权去进行failover.
**redis sentinel保证了安全性**：每个试图去failover同一个master的sentinel都会得到一个独一无二的版本号。



## 配置传播

- 一旦一个sentinel成功地对一个master进行了failover，它将会把关于master的最新配置通过广播形式通知其它sentinel，其它的sentinel则更新对应master的配置。
- 一个faiover要想被成功实行，sentinel必须能够向选为master的slave发送`SLAVE OF NO ONE`命令，然后能够通过`INFO`命令看到新master的配置信息。
- 当将一个slave选举为master并发送 `SLAVE OF NO ONE`后，即使其它的slave还没针对新master重新配置自己，failover也被认为是成功了的，然后所有sentinels将会发布新的配置信息。
- 新配在集群中相互传播的方式，就是为什么我们需要当一个sentinel进行failover时必须被授权一个版本号的原因。
- 每个sentinel使用 **发布/订阅** 的方式持续地传播master的配置版本信息，配置传播的 **发布/订阅** 管道是：`__sentinel__:hello`。
- 因为每一个配置都有一个版本号，所以以版本号最大的那个为标准。

举个栗子：假设有一个名为mymaster的地址为192.168.1.50:6379。一开始，集群中所有的sentinel都知道这个地址，于是为mymaster的配置打上版本号1。一段时候后mymaster死了，有一个sentinel被授权用版本号2对其进行failover。如果failover成功了，假设地址改为了192.168.1.50:9000，此时配置的版本号为2，进行failover的sentinel会将新配置广播给其他的sentinel，由于其他sentinel维护的版本号为1，发现新配置的版本号为2时，版本号变大了，说明配置更新了，于是就会采用最新的版本号为2的配置。

这意味着sentinel集群保证了第二种活跃性：一个能够互相通信的sentinel集群最终会采用版本号最高且相同的配置。



## SDOWN和ODOWN的更多细节

- sentinel对于**不可用**有两种不同的看法，一个叫**主观不可用**(SDOWN),另外一个叫**客观不可用**(ODOWN)。SDOWN是sentinel自己主观上检测到的关于master的状态，ODOWN需要一定数量的sentinel达成一致意见才能认为一个master客观上已经宕掉，各个sentinel之间通过命令`SENTINEL is_master_down_by_addr`来获得其它sentinel对master的检测结果。
- 从sentinel的角度来看，如果发送了**PING**心跳后，在一定时间内没有收到合法的回复，就达到了SDOWN的条件。这个时间在配置中通过`is-master-down-after-milliseconds`参数配置。
- 当sentinel发送**PING**后，以下回复之一都被认为是合法的：

```
PING replied with +PONG.
PING replied with -LOADING error.
PING replied with -MASTERDOWN error.

```

- 其它任何回复（或者根本没有回复）都是不合法的。
- 从SDOWN切换到ODOWN不需要任何一致性算法，只需要一个gossip协议：如果一个sentinel收到了足够多的sentinel发来消息告诉它某个master已经down掉了，SDOWN状态就会变成ODOWN状态。如果之后master可用了，这个状态就会相应地被清理掉。
- 正如之前已经解释过了，真正进行failover需要一个授权的过程，但是所有的failover都开始于一个ODOWN状态。
- ODOWN状态只适用于master，对于不是master的redis节点sentinel之间不需要任何协商，slaves和sentinel不会有ODOWN状态。



## Sentinel之间和Slaves之间的自动发现机制

- 虽然sentinel集群中各个sentinel都互相连接彼此来检查对方的可用性以及互相发送消息。但是你不用在任何一个sentinel配置任何其它的sentinel的节点。因为sentinel利用了master的**发布/订阅**机制去自动发现其它也监控了统一master的sentinel节点。
- 通过向名为`__sentinel__:hello`的管道中发送消息来实现。
- 同样，你也不需要在sentinel中配置某个master的所有slave的地址，sentinel会通过询问master来得到这些slave的地址的。
- 每个sentinel通过向每个master和slave的**发布/订阅**频道`__sentinel__:hello`每秒发送一次消息，来宣布它的存在。
- 每个sentinel也订阅了每个master和slave的频道`__sentinel__:hello`的内容，来发现未知的sentinel，当检测到了新的sentinel，则将其加入到自身维护的master监控列表中。
- 每个sentinel发送的消息中也包含了其当前维护的最新的master配置。如果某个sentinel发现自己的配置版本低于接收到的配置版本，则会用新的配置更新自己的master配置。
- 在为一个master添加一个新的sentinel前，sentinel总是检查是否已经有sentinel与新的sentinel的进程号或者是地址是一样的。如果是那样，这个sentinel将会被删除，而把新的sentinel添加上去。