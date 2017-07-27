---
categories: Redis
date: 2017-07-17 10:00
title: Redis 之 持久化与复制
---

Redis 提供了两种不同的持久化方法来将数据存储到硬盘里面。`快照（snapshotting）`和 `只追加文件（append-only file, AOF）`，它会在执行写命令时，将被执行的写命令复制到硬盘里面。

<!-- more -->

```properties
## 快照持久化选项
save 60 1000
stop-writes-on-bgsave-error no
rdbcompression yes
dbfilename dump.rdb

## AOF持久化选项
appendonly no
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

## 共享选项，决定了快照和AOF文件的保存位置
dir ./
```



# 1 持久化选项

## 1.1 快照持久化

创建快照的几种办法

1. 向Redis发送BGSAVE命令来创建一个快照。Redis会调用fork命令来创建一个子进程负责将快照写进硬盘。
2. 向Redis发送SAVE命令来创建一个快照。在快照创建完毕之前将不再相应任何其他命令。
3. 设置了save选项，如“save 60 10000”，那么从Redis最近一次创建快照之后开始算起，当“60秒之内有10000次写入”这个条件满足时，Redis就会自动触发BGSAVE命令。若有多个选项，任意一个满足即触发。
4. 当Redis接受到SHUTDOWN命令，或者标准的TERM信号时，会执行SAVE命令。
5. 当一个Redis服务器连接另外一个Redis服务器，并向对方发送SYNC命令来开始一次复制操作时，如果主服务器目前没有在执行BGSAVE操作，或者并非刚刚执行完BGSAVE操作。




## 1.2 AOF持久化

- AOF持久化会将被执行的写命令写到AOF文件的末尾，以此来记录数据发生的变化。
- AOF持久化可以通过设置`appendonly yes`配置选项来打开。
- appendfsync选项及同步频率，`appendfsync` 有 `always`、`everysec`、`no` 三个选项
  - `always`：每个写命令都要同步写入硬盘。这样做会严重降低Redis的速度
  - `everysec`：每秒执行以此同步，显示地将多个写命令同步到硬盘
  - `no`：让操作系统来决定应该何时进行同步
  - 注：Redis每秒同步以此AOF文件时的性能和不使用任何持久化特性时的性能相差无几。




## 1.3 重写/压缩AOF文件

随着Redis不断运行，AOF文件的体积也会不断增长，在极端情况下，体积不断增大的AOF文件甚至可能会用完硬盘的所有可用空间。另外，因为Redis重启之后需要通过重新执行AOF文件记录的所有写命令来还原数据集，所以如果AOF文件的体积非常大，那么还原操作执行的时间就可能会非常长。

- 用户可以向Redis发送`BGREWRITEAOF` 命令来移除AOF文件中的冗余命令来重写（rewrite）AOF文件，使文件体积尽可能小。
- AOF持久化可以通过设置`auto-aof-rewrite-percentage`选项和`auto-aof-rewrite-min-size`选项来自动执行 `BGREWRITEAOF`。如：设置了auto-aof-rewrite-percentage 100 、auto-aof-rewrite-min-size 64mb，并且启用了AOF持久化，那么当AOF文件的体积大于64MB，并且AOF文件的体积比上一次重写之后的体积大了至少一倍（100%）的时候，Redis将执行`BGREWRITEAOF`




# 2 复制

- 复制可以让其他服务器拥有一个不断更新的数据副本，从而使得拥有数据副本的服务器可以用于处理客户端发送的读请求。
- 从服务器在接收到主服务器发送的数据初始副本（initial copy of the data）之后，客户端每次向主服务器进行写入时，从服务器都会实时地得到更新。
- 在部署好主从服务器之后，客户端就可以向任意一个从服务区发送读请求了。（客户端通常会随机地选择使用哪个从服务器，从而将负载平均分配到各个从服务器上）。




## 2.1 对Redis的复制相关选项进行设置

开启从服务器所必须的选项只有`slaveof` 一个。

1. 配置文件中包含 `slaveof host port` 让 Redis 服务器启动时成为另外一个服务器的从服务器。
2. 运行时发送 `slaveof host port` 或 `slaveof no one` 命令来让从服务器开始或终止复制操作。 



## 2.2 Redis 复制的启动过程

| 步骤   | 主服务器操作                                   | 从服务器操作                                   |
| ---- | ---------------------------------------- | ---------------------------------------- |
| 1    | （等待命令进入）                                 | 连接（或者重连接主服务器，发送`SYNC`命令）                 |
| 2    | 开始执行`BGSAVE`，并使用缓冲区记录`BGSAVE`之后执行的所有写命令  | 根据配置选项来决定是继续使用现有的数据（如果有的话）来处理客户端的命令请求，还是向发送请求的客户端返回错误 |
| 3    | `BGSAVE` 执行完毕，向从服务器发送快照文件，并在发送期间继续使用缓冲区记录被执行的写命令 | 丢弃所有旧数据（如果有的话），开始载入主服务器发来的快照文件           |
| 4    | 快照文件发送完毕，开始向从服务器发送存储在缓冲区里面的写命令           | 完成对快照文件的解释操作，像往常一样开始接受命令请求               |
| 5    | 缓冲区存储的写命令发送完毕；从现在开始，每执行一个写命令，就向从服务器发送相同的写命令 | 执行主服务器发送来的所有存储在缓冲区里面的写命令；并从现在开始，接收并执行主服务器传来的每个写命令 |

实际中最好还是让主服务器只使用50%~65%的内存，留下30%~45%的内存用于执行`BGSAVE`命令和创建记录写命令的缓冲区。



## 2.3 Redis 设置主从服务器的具体操作

### 2.3.1 简单地先复制一份Redis文件。

```shell
# 复制一份文件作为从服务器
cp redis-master redis-slave
```



### 2.3.2 修改从服务器配置文件

配置文件 redis-slave/redis.conf（和redis master差异的地方）

```shell
# 修改监听端口
port 6380

# 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
# 也可以在从服务器运行时执行以下命令，实现同样的效果
slaveof 127.0.0.1 6379
```


### 2.3.3 启动主从服务器

现在先采用运行时设置从服务器的方法，启动主从服务器

```shell
# 启动主服务器
redis-master/src/redis-server redis-master/redis.conf

# 启动从服务器
redis-slave/src/redis-server redis-slave/redis.conf
```



### 2.3.4 检测同步前的数据

为主服务器设置测试数据

```shell
127.0.0.1:6379> set test test
OK
127.0.0.1:6379> get test
"test"
127.0.0.1:6379> 
```

检测从服务器是否有相同的数据

```shell
127.0.0.1:6380> get test
(nil)
127.0.0.1:6380> 
```

可以发现现在从服务器并没有同步到主服务器的数据



### 2.3.5 运行时设置从服务器

现在对从服务器执行

```shell
slaveof 127.0.0.1 6379
```



此时可发现主服务器控制台输出了以下日志

```shell
1078:M 16 Jul 15:35:49.470 * Slave 127.0.0.1:6380 asks for synchronization
1078:M 16 Jul 15:35:49.470 * Full resync requested by slave 127.0.0.1:6380
1078:M 16 Jul 15:35:49.470 * Starting BGSAVE for SYNC with target: disk
1078:M 16 Jul 15:35:49.471 * Background saving started by pid 1150
1150:C 16 Jul 15:35:49.473 * DB saved on disk
1078:M 16 Jul 15:35:49.574 * Background saving terminated with success
1078:M 16 Jul 15:35:49.574 * Synchronization with slave 127.0.0.1:6380 succeeded
```

从服务区控制台输出日志

```shell
1090:S 16 Jul 15:35:49.042 * SLAVE OF 127.0.0.1:6379 enabled (user request from 'id=2 addr=127.0.0.1:50602 fd=6 name= age=231 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=slaveof')
1090:S 16 Jul 15:35:49.469 * Connecting to MASTER 127.0.0.1:6379
1090:S 16 Jul 15:35:49.469 * MASTER <-> SLAVE sync started
1090:S 16 Jul 15:35:49.469 * Non blocking connect for SYNC fired the event.
1090:S 16 Jul 15:35:49.469 * Master replied to PING, replication can continue...
1090:S 16 Jul 15:35:49.470 * Partial resynchronization not possible (no cached master)
1090:S 16 Jul 15:35:49.471 * Full resync from master: 858ddf1bd0661c9f0a29c91a0d7f5ea0f59a9e23:1
1090:S 16 Jul 15:35:49.574 * MASTER <-> SLAVE sync: receiving 31 bytes from master
1090:S 16 Jul 15:35:49.574 * MASTER <-> SLAVE sync: Flushing old data
1090:S 16 Jul 15:35:49.574 * MASTER <-> SLAVE sync: Loading DB in memory
1090:S 16 Jul 15:35:49.575 * MASTER <-> SLAVE sync: Finished with success
```

可以看到主服务器在接收到从服务器的同步请求之后，执行了`BGSAVE`。



### 2.3.6 再次检查同步后的数据

测试从服务器是否同步到了主服务器的旧数据

```shell
127.0.0.1:6380> get test
"test"
127.0.0.1:6380> 
```

可以看到从服务器已经拥有了和主服务器一样的数据



测试从服务器是否能实时同步主服务器的数据变化

```shell
127.0.0.1:6379> set dev dev
OK
127.0.0.1:6379> 
```

```shell
127.0.0.1:6380> get dev
"dev"
127.0.0.1:6380> 
```

当尝试向从服务区写命令的时候会出现

```shell
(error) READONLY You can't write against a read only slave.
```

**注：**slaveof参数设置需要同步的master服务器，slave-read-only设置是否可写。如果 slave-read-only 值为 no 的时候，在从上进行写操作（如set key1 1111），会提示“(error) READONLY You can’t write against a read only slave.”、



## 2.4 主从链

创建多个从服务器可能会造成网络的不可用——当复制需要通过互联网进行或者需要在不同数据中心之间进行时，尤为如此。可以通过为从服务器设置从服务器，并由此形成`主从链（master/slave chaining）` 来分担主服务器的复制工作。

从服务器对从服务器进行复制操作和从服务器对主服务器进行复制操作唯一的区别在于，如果从服务器X拥有从服务器Y，那么当从服务器X在执行步骤4，即解释快照文件时，它将断开与从服务器Y的连接，导致从服务器Y需要重新连接并重新同步（resync）。

