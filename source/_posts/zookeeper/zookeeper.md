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




### 伪集群模式

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

- **启动集群 zkStartAll.sh**

```
#!/usr/bin/env bash

./bin/zkServer.sh start ./server/zk1/conf/zoo.cfg
./bin/zkServer.sh start ./server/zk2/conf/zoo.cfg
./bin/zkServer.sh start ./server/zk3/conf/zoo.cfg
```

- **停止集群 zkStopAll.sh**

```
#!/usr/bin/env bash

./bin/zkServer.sh stop ./server/zk1/conf/zoo.cfg
./bin/zkServer.sh stop ./server/zk2/conf/zoo.cfg
./bin/zkServer.sh stop ./server/zk3/conf/zoo.cfg
```



4. **启动集群**

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

**检查是否启动**

```
$ jps
1440 Jps
1425 QuorumPeerMain
1413 QuorumPeerMain
1437 QuorumPeerMain
```

**查看服务器状态**

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

**注：可以看到 zk2 c成为了集群的 leader。**



5. **停止集群**

   ```
   $ ./zkStopAll.sh 

   ZooKeeper JMX enabled by default
   Using config: ./server/zk1/conf/zoo.cfg
   Stopping zookeeper ... STOPPED
   ZooKeeper JMX enabled by default
   Using config: ./server/zk2/conf/zoo.cfg
   Stopping zookeeper ... STOPPED
   ZooKeeper JMX enabled by default
   Using config: ./server/zk3/conf/zoo.cfg
   Stopping zookeeper ... STOPPED
   ```

   ​



### 集群模式

集群模式和伪集群模式很类似，详细参考文后的“参考文章”。





## 客户端

### 连接服务器

在服务端开启的情况下，运行客户端，使用如下命令：**./zkCli.sh**

```
$ ./bin/zkCli.sh 

Connecting to localhost:2181
2017-08-12 10:09:16,917 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
2017-08-12 10:09:16,920 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=172.20.10.7
2017-08-12 10:09:16,920 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_131
2017-08-12 10:09:16,922 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
2017-08-12 10:09:16,922 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/Users/cjz/Documents/devtool/zookeeper/bin/../build/classes:/Users/cjz/Documents/devtool/zookeeper/bin/../build/lib/*.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../lib/netty-3.10.5.Final.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../lib/log4j-1.2.16.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../lib/jline-0.9.94.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../zookeeper-3.4.10.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../src/java/lib/*.jar:/Users/cjz/Documents/devtool/zookeeper/bin/../conf:
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/Users/cjz/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/var/folders/dj/040kxgk101q1mf73mv38wdt00000gn/T/
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Mac OS X
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=x86_64
2017-08-12 10:09:16,923 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=10.12.5
2017-08-12 10:09:16,924 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=cjz
2017-08-12 10:09:16,924 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/Users/cjz
2017-08-12 10:09:16,924 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/Users/cjz/Documents/devtool/zookeeper
2017-08-12 10:09:16,925 [myid:] - INFO  [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@531d72ca
Welcome to ZooKeeper!
2017-08-12 10:09:16,950 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/127.0.0.1:2181. Will not attempt to authenticate using SASL (unknown error)
JLine support is enabled
2017-08-12 10:09:17,034 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@876] - Socket connection established to localhost/127.0.0.1:2181, initiating session
2017-08-12 10:09:17,064 [myid:] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/127.0.0.1:2181, sessionid = 0x15dd433e0e30000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] 
```



> zookeeper默认的客户端连接`端口为 2181` ，若想连接不同的主机或不同端口的服务器，可使用如下命令：**./bin/zkCli.sh -server ip:port**
>
> 如： ./bin//zkCli.sh -server 127.0.0.1:2182



### 创建节点

使用create命令，可以创建一个Zookeeper节点， 如

```
create [-s][-e] path data acl
```

其中，-s 或 -e 分别指定节点特性，-s 指定顺序节点，-e 指示临时节点，若不指定，则表示持久节点；acl用来进行权限控制。

1. **创建永久节点**

使用 **create /zk-permanent 123** 命令创建zk-permanent永久节点

```
[zk: localhost:2181(CONNECTED) 0] create /zk-permanent 123
Created /zk-permanent
[zk: localhost:2181(CONNECTED) 1] ls /
[zk-permanent, zookeeper]
```



2. **创建临时节点**

- 使用 **create -e /zk-temp 123** 命令创建zk-temp临时节点

```
[zk: localhost:2181(CONNECTED) 2] create -e /zk-temp 123
Created /zk-temp
[zk: localhost:2181(CONNECTED) 3] ls /
[zk-permanent, zookeeper, zk-temp]
```

- 临时节点在客户端会话结束后，就会自动删除，下面使用**quit**命令退出客户端

```
[zk: localhost:2181(CONNECTED) 4] quit
Quitting...
2017-08-12 10:34:59,283 [myid:] - INFO  [main:ZooKeeper@684] - Session: 0x15dd433e0e30001 closed
2017-08-12 10:34:59,286 [myid:] - INFO  [main-EventThread:ClientCnxn$EventThread@519] - EventThread shut down for session: 0x15dd433e0e30001
```

- 再次使用客户端连接服务端，并使用ls / 命令查看根目录下的节点

```
[zk: localhost:2181(CONNECTED) 0] ls /
[k-permanent, zookeeper]
```

可以看到根目录下已经不存在zk-temp临时节点了。



3. **创建顺序节点**

使用 **create -s /zk-test 123** 命令创建zk-test顺序节点

```
[zk: localhost:2181(CONNECTED) 1] create -s /zk-test 123
Created /zk-test0000000005
[zk: localhost:2181(CONNECTED) 2] ls / 
[zk-permanent, zookeeper, zk-test0000000005]
```

可以看到创建的zk-test节点后面添加了一串数字以示区别。



### 读取节点

与读取相关的命令有ls 命令和get 命令，ls命令可以列出Zookeeper指定节点下的所有子节点，只能查看指定节点下的第一级的所有子节点；get命令可以获取Zookeeper指定节点的数据内容和属性信息。其用法分别如下

**ls path [watch]**

**get path [watch]**

**ls2 path [watch]**



- 若获取根节点下面的所有子节点，使用**ls /** 命令即可

```
[zk: localhost:2181(CONNECTED) 2] ls / 
[zk-permanent, zookeeper, zk-test0000000005]
```



- 若想获取根节点数据内容和属性信息，使用**get /** 命令即可

```
[zk: localhost:2181(CONNECTED) 3] get /zk-permanent
123
cZxid = 0x600000006
ctime = Sat Aug 12 10:30:23 CST 2017
mZxid = 0x600000006
mtime = Sat Aug 12 10:30:23 CST 2017
pZxid = 0x600000006
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```



- 也可以使用**ls2 /** 命令同时查看节点下的面的所有子节点，并获取节点数据内容和属性信息

```
[zk: localhost:2181(CONNECTED) 5] ls2 /
[zk-permanent, zookeeper, zk-test0000000005]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x60000000a
cversion = 8
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 3
```



### 更新节点

使用set命令，可以更新指定节点的数据内容，用法如下

**set path data [version]**

其中，data就是要更新的新内容，version表示数据版本，



- 将/zk-permanent节点的数据更新为456，可以使用如下命令：**set /zk-permanent 456**

```
[zk: localhost:2181(CONNECTED) 6] set /zk-permanent 456
cZxid = 0x600000006
ctime = Sat Aug 12 10:30:23 CST 2017
mZxid = 0x60000000b
mtime = Sat Aug 12 10:47:55 CST 2017
pZxid = 0x600000006
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```

现在dataVersion已经变为1了，表示进行了更新。



- 更新数据时可以加入版本号验证

  尝试用错误的版本号更新节点数据会提示 **版本号非法 version No is not valid** 

```
[zk: localhost:2181(CONNECTED) 8] set /zk-permanent 789 0 
version No is not valid : /zk-permanent
[zk: localhost:2181(CONNECTED) 9] set /zk-permanent 789 1 
cZxid = 0x600000006
ctime = Sat Aug 12 10:30:23 CST 2017
mZxid = 0x60000000e
mtime = Sat Aug 12 10:49:26 CST 2017
pZxid = 0x600000006
cversion = 0
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
[zk: localhost:2181(CONNECTED) 10] 
```



- 不使用版本号以及使用 -1 作为版本号可以跳过版本校验

```
[zk: localhost:2181(CONNECTED) 10] set /zk-permanent 123 -1
cZxid = 0x600000006
ctime = Sat Aug 12 10:30:23 CST 2017
mZxid = 0x60000000f
mtime = Sat Aug 12 10:51:52 CST 2017
pZxid = 0x600000006
cversion = 0
dataVersion = 3
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```



### 删除节点

使用delete命令可以删除Zookeeper上的指定节点，用法如下

**delete path [version]**

其中version也是表示数据版本



使用**delete /zk-permanent** 命令即可删除/zk-permanent节点

```
[zk: localhost:2181(CONNECTED) 11] delete /zk-permanent
[zk: localhost:2181(CONNECTED) 12] ls /
[zookeeper, zk-test0000000005]
```

可以看到，已经成功删除/zk-permanent节点。值得注意的是，**若删除节点存在子节点，那么无法删除该节点，必须先删除子节点，再删除父节点。**



参考文章:

[【分布式】Zookeeper使用--命令行](http://www.cnblogs.com/leesf456/p/6022357.html)

[【Zookeeper系列一】Zookeeper应用介绍与安装部署](https://my.oschina.net/xianggao/blog/531204)

《从Paxos到Zookeeper分布式一致性原理与实践》