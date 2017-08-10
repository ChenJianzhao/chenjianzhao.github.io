---
categories: zookeeper
date: 2017-08-09 22:00
title: Zookeeper 命令行
---



## Zookeeper部署

Zookeeper有三种运行形式：**集群模式、单机模式、伪集群模式**。

<!-- more -->



**bin目录下常用的脚本解释**

zkCleanup　　清理Zookeeper历史数据，包括食物日志文件和快照数据文件

zkCli　　　　  Zookeeper的一个简易客户端

zkEnv　　　　设置Zookeeper的环境变量

zkServer　　   Zookeeper服务器的启动、停止、和重启脚本



### 单机模式

ZooKeeper的单机模式通常是用来快速测试客户端应用程序的，在实际过程中不可能是单机模式。单机模式的配置也比较简单。

1. **编写配置文件zoo.cfg**

   ZooKeeper的运行**默认是读取zoo.cfg文件**里面的内容的，需要从 `zoo_sample.cfg` 复制一份进行修改：

   ```
   tickTime=2000
   initLimit=10
   syncLimit=5
   dataDir=./server/zk1/data
   clientPort=2181
   ```

   我们需要**指定 dataDir 的值，它指向了一个目录，这个目录在开始的时候需要为空**。

   下面是每个参数的含义：

   **tickTime** ：基本事件单元，以毫秒为单位。这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。

   **dataDir** ：存储内存中数据库快照的位置，顾名思义就是 Zookeeper 保存数据的目录，**默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里**。

   **clientPort** ：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

   使用单机模式时用户需要注意：这种配置方式下没有 ZooKeeper 副本，所以如果 ZooKeeper 服务器出现故障， ZooKeeper 服务将会停止。

   ​

2. **启动Zookeeper**

   ```
   $ ./bin/zkServer.sh start zoo.cfg
   ZooKeeper JMX enabled by default
   Using config: ./server/zk1/conf/zoo.cfg
   Starting zookeeper ... STARTED
   ```

   **zk的服务显示为QuorumPeerMain**：

   ```
   $ jps
   5321 QuorumPeerMain
   5338 Jps
   ```

   **查看运行状态**：

   ```
   $ ./bin/zkServer.sh status zoo.cfg
   JMX enabled by default
   Using config: ./server/zk1/conf/zoo.cfg
   Mode: standalone
   ```

   **单节点的时，Mode会显示为standalone。**

   ​

3. **停止ZooKeeper服务**

   ```
   $ ./bin/zkServer.sh stop zoo.cfg
   JMX enabled by default
   Using config: ./server/zk1/conf/zoo.cfg
   Stopping zookeeper ... STOPPED
   ```



## 伪集群模式

所谓 “伪分布式集群” 就是在，在一台PC中，启动多个ZooKeeper的实例。“完全分布式集群” 是每台PC，启动一个ZooKeeper实例。其实在企业中式不会存在的，另外为了测试一个客户端程序也没有必要存在，只有在物质条件比较匮乏的条件下才会存在的模式。集群伪分布模式就是在单机下模拟集群的ZooKeeper服务，在一台机器上面有多个ZooKeeper的JVM同时运行。

ZooKeeper的集群模式下，**多个Zookeeper服务器在工作前会选举出一个Leader**，在接下来的工作中这个被选举出来的Leader死了，而剩下的Zookeeper服务器会知道这个Leader死掉了，**在活着的Zookeeper集群中会继续选出一个Leader**，选举出Leader的目的是为了可以在分布式的环境中保证数据的一致性。



1. **确认集群服务器的数量**

   由于ZooKeeper集群中，会有一个Leader负责管理和协调其他集群服务器，**因此服务器的数量通常都是单数**，例如3，5，7...等，**这样2n+1的数量的服务器就可以允许最多n台服务器的失效**。

   ​

2. **创建环境目录**

   ```
   ~ mkdir ./server/zk1/data
   ~ mkdir ./server/zk2/data
   ~ mkdir ./server/zk3/data

   #新建myid文件
   ~ echo "1" > ./server/zk1/data/myid
   ~ echo "2" > ./server/zk2/data/myid
   ~ echo "3" > ./server/zk3/data/myid
   ```

   ​

3. **分别修改配置文件**

   `修改`：dataDir,clientPort

   `增加`：集群的实例，server.X，”X”表示每个目录中的myid的值

   ```
   ~ vim ./server/zk1/conf/zoo.cfg
   tickTime=2000
   initLimit=10
   syncLimit=5
   dataDir=/home/taomk/zk/zoo/zk1
   clientPort=2181
   server.1=127.0.0.1:2888:3888
   server.2=127.0.0.1:2889:3889
   server.3=127.0.0.1:2890:3890

   ~ vim ./server/zk2/conf/zoo.cfg
   tickTime=2000
   initLimit=10
   syncLimit=5
   dataDir=/home/taomk/zk/zoo/zk2
   clientPort=2182
   server.1=127.0.0.1:2888:3888
   server.2=127.0.0.1:2889:3889
   server.3=127.0.0.1:2890:3890

   ~ vim ./server/zk3/conf/zoo.cfg
   tickTime=2000
   initLimit=10
   syncLimit=5
   dataDir=/home/taomk/zk/zoo/zk3
   clientPort=2183
   server.1=127.0.0.1:2888:3888
   server.2=127.0.0.1:2889:3889
   server.3=127.0.0.1:2890:3890
   ```



**为了操作简便，可以编写一个简单的启动、停止多个服务器的脚本**

zkStartAll.sh

```
#!/usr/bin/env bash

./bin/zkServer.sh start ./server/zk1/conf/zoo.cfg
./bin/zkServer.sh start ./server/zk2/conf/zoo.cfg
./bin/zkServer.sh start ./server/zk3/conf/zoo.cfg
```

zkStopAll.sh

```
#!/usr/bin/env bash

./bin/zkServer.sh stop ./server/zk1/conf/zoo.cfg
./bin/zkServer.sh stop ./server/zk2/conf/zoo.cfg
./bin/zkServer.sh stop ./server/zk3/conf/zoo.cfg
```



**启动集群**

```shell
$ ./zkStartAll.sh 
ZooKeeper JMX enabled by default
Using config: ./server/zk1/conf/zoo.cfg
Starting zookeeper ... STARTED
ZooKeeper JMX enabled by default
Using config: ./server/zk2/conf/zoo.cfg
Starting zookeeper ... STARTED
ZooKeeper JMX enabled by default
Using config: ./server/zk3/conf/zoo.cfg
Starting zookeeper ... STARTED
```

检查是否启动

```
$ jps
1440 Jps
1425 QuorumPeerMain
1413 QuorumPeerMain
1437 QuorumPeerMain
```

查看服务器状态

```
$ ./bin/zkServer.sh status server/zk1/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk1/conf/zoo.cfg
Mode: follower

$ ./bin/zkServer.sh status server/zk2/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk2/conf/zoo.cfg
Mode: leader

$ ./bin/zkServer.sh status server/zk3/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk3/conf/zoo.cfg
Mode: follower
```

可以看到 zk2 称为了 leader