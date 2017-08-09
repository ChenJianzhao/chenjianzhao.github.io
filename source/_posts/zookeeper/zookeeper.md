---
categories: zookeeper
date: 2017-08-09 22:00
title: Zookeeper 服务端启动
---



<!-- more -->

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



启动服务器

```shell
chenjianzhaodeMacBook-Pro:zookeeper cjz$ ./zkStartAll.sh 
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
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps
1440 Jps
1425 QuorumPeerMain
1413 QuorumPeerMain
1437 QuorumPeerMain
```



查看服务器状态

```
chenjianzhaodeMacBook-Pro:zookeeper cjz$ ./bin/zkServer.sh status server/zk1/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk1/conf/zoo.cfg
Mode: follower

chenjianzhaodeMacBook-Pro:zookeeper cjz$ ./bin/zkServer.sh status server/zk2/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk2/conf/zoo.cfg
Mode: leader

chenjianzhaodeMacBook-Pro:zookeeper cjz$ ./bin/zkServer.sh status server/zk3/conf/zoo.cfg 
ZooKeeper JMX enabled by default
Using config: server/zk3/conf/zoo.cfg
Mode: follower
```

可以看到 zk2 称为了 leader