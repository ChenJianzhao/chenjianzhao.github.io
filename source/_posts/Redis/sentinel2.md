---
categories: Redis
date: 2017-07-18 20:10
title: Redis 之 Sentinel实操
---

<!-- more -->

# 启动Redis集群

## 启动1个master，2个slave

**master 日志**

```
[3368] 18 Jul 09:49:34.648 # Server started, Redis version 2.6.12
[3368] 18 Jul 09:49:34.938 * DB loaded from disk: 0.290 seconds
[3368] 18 Jul 09:49:34.938 * The server is now ready to accept connections on port 6379
[7316] 18 Jul 09:54:25.569 # Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 7316
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               
              
[7316] 18 Jul 09:54:25.569 # Server started, Redis version 2.6.12
[7316] 18 Jul 09:54:25.839 * DB loaded from disk: 0.270 seconds
[7316] 18 Jul 09:54:25.839 * The server is now ready to accept connections on port 6379
[7316] 18 Jul 09:56:31.730 * Slave ask for synchronization
[7316] 18 Jul 09:56:31.730 * Starting BGSAVE for SYNC
[7316] 18 Jul 09:56:31.730 * cowBkgdSaveReset deleting 0 SDS and 0 obj items
[7316] 18 Jul 09:56:31.980 * DB saved on disk
[7316] 18 Jul 09:56:32.030 * Background saving terminated with success
[7316] 18 Jul 09:56:32.030 * cowBkgdSaveReset deleting 0 SDS and 1 obj items
[7316] 18 Jul 09:56:32.070 * Synchronization with slave succeeded
[7316] 18 Jul 10:01:18.638 * Slave ask for synchronization
[7316] 18 Jul 10:01:18.638 * Starting BGSAVE for SYNC
[7316] 18 Jul 10:01:18.638 * cowBkgdSaveReset deleting 0 SDS and 0 obj items
[7316] 18 Jul 10:01:18.868 * DB saved on disk
[7316] 18 Jul 10:01:18.888 * Background saving terminated with success
[7316] 18 Jul 10:01:18.888 * cowBkgdSaveReset deleting 0 SDS and 1 obj items
[7316] 18 Jul 10:01:18.938 * Synchronization with slave succeeded
```



**slave1 日志**

```
[2040] 18 Jul 09:56:30.705 # Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6380
 |    `-._   `._    /     _.-'    |     PID: 2040
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

[2040] 18 Jul 09:56:30.705 # Server started, Redis version 2.6.12
[2040] 18 Jul 09:56:30.981 * DB loaded from disk: 0.276 seconds
[2040] 18 Jul 09:56:30.981 * The server is now ready to accept connections on port 6380
[2040] 18 Jul 09:56:31.710 * Connecting to MASTER...
[2040] 18 Jul 09:56:31.730 * MASTER <-> SLAVE sync started
[2040] 18 Jul 09:56:31.730 * Non blocking connect for SYNC fired the event.
[2040] 18 Jul 09:56:31.730 * Master replied to PING, replication can continue...
[2040] 18 Jul 09:56:32.030 * MASTER <-> SLAVE sync: receiving 1472727 bytes from master
[2040] 18 Jul 09:56:32.070 * MASTER <-> SLAVE sync: Loading DB in memory
[2040] 18 Jul 09:56:32.480 * MASTER <-> SLAVE sync: Finished with success
```



**slave2 日志**

```
[976] 18 Jul 10:01:17.627 # Warning: 32 bit instance detected but no memory limit set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6381
 |    `-._   `._    /     _.-'    |     PID: 976
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

[976] 18 Jul 10:01:17.627 # Server started, Redis version 2.6.12
[976] 18 Jul 10:01:17.897 * DB loaded from disk: 0.270 seconds
[976] 18 Jul 10:01:17.897 * The server is now ready to accept connections on port 6381
[976] 18 Jul 10:01:18.628 * Connecting to MASTER...
[976] 18 Jul 10:01:18.628 * MASTER <-> SLAVE sync started
[976] 18 Jul 10:01:18.628 * Non blocking connect for SYNC fired the event.
[976] 18 Jul 10:01:18.628 * Master replied to PING, replication can continue...
[976] 18 Jul 10:01:18.888 * MASTER <-> SLAVE sync: receiving 1472736 bytes from master
[976] 18 Jul 10:01:18.938 * MASTER <-> SLAVE sync: Loading DB in memory
[976] 18 Jul 10:01:19.338 * MASTER <-> SLAVE sync: Finished with success
```



可以看到2个slave启动后向master请求同步。



## 分别启动3个sentinel

**sentinel1 日志**

```
[6876] 18 Jul 10:22:26.690 # Warning: 32 bit instance detected but no memory lim
it set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 6876
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[6876] 18 Jul 10:22:27.700 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6876] 18 Jul 10:22:27.700 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6876] 18 Jul 10:23:48.375 * +sentinel sentinel 127.0.0.1:26380 127.0.0.1 26380@ mymaster 127.0.0.1 6379
[6876] 18 Jul 10:29:14.747 * +sentinel sentinel 127.0.0.1:26381 127.0.0.1 26381@ mymaster 127.0.0.1 6379
```



**sentinel2 日志**

```
[4620] 18 Jul 10:23:43.361 # Warning: 32 bit instance detected but no memory lim
it set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26380
 |    `-._   `._    /     _.-'    |     PID: 4620
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[4620] 18 Jul 10:23:44.368 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[4620] 18 Jul 10:23:44.368 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[4620] 18 Jul 10:23:47.515 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6379
[4620] 18 Jul 10:29:14.747 * +sentinel sentinel 127.0.0.1:26381 127.0.0.1 26381 @ mymaster 127.0.0.1 6379
```



**sentinel3 日志**

```
[6612] 18 Jul 10:29:09.713 # Warning: 32 bit instance detected but no memory lim
it set. Setting 3 GB maxmemory limit with 'noeviction' policy now.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 2.6.12 (00000000/0) 32 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26381
 |    `-._   `._    /     _.-'    |     PID: 6612
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[6612] 18 Jul 10:29:09.723 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6612] 18 Jul 10:29:09.723 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6612] 18 Jul 10:29:10.595 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6379
[6612] 18 Jul 10:29:10.997 * +sentinel sentinel 127.0.0.1:26380 127.0.0.1 26380 @ mymaster 127.0.0.1 6379
```



## 通过 sentinel API 查看 master 的信息   

- **SENTINEL masters**  查看所有监控的master的信息

```
redis 127.0.0.1:26379> SENTINEL masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6379"
    7) "runid"
    8) "01c0a31c597c4530dae32773f5fd85949dd0291e"
    9) "flags"
   10) "master"
   11) "pending-commands"
   12) "0"
   13) "last-ok-ping-reply"
   14) "250"
   15) "last-ping-reply"
   16) "250"
   17) "info-refresh"
   18) "5169"
   19) "num-slaves"
   20) "2"
   21) "num-other-sentinels"
   22) "2"
   23) "quorum"
   24) "2"
```



- **SENTINEL slaves mymaster** 查看指定master的所有slave 的信息

```
redis 127.0.0.1:26379> SENTINEL slaves mymaster
1)  1) "name"
    2) "127.0.0.1:6381"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6381"
    7) "runid"
    8) "ecc7cd52da8edf83d9f647edebae6aec6a3e5ff7"
    9) "flags"
   10) "slave"
   11) "pending-commands"
   12) "0"
   13) "last-ok-ping-reply"
   14) "563"
   15) "last-ping-reply"
   16) "563"
   17) "info-refresh"
   18) "2566"
   19) "master-link-down-time"
   20) "0"
   21) "master-link-status"
   22) "ok"
   23) "master-host"
   24) "localhost"
   25) "master-port"
   26) "6379"
   27) "slave-priority"
   28) "100"
2)  1) "name"
    2) "127.0.0.1:6380"
    3) "ip"
    4) "127.0.0.1"
    5) "port"
    6) "6380"
    7) "runid"
    8) "41ee511e94c83b5111dc135c4eafe9ffb52f3d18"
    9) "flags"
   10) "slave"
   11) "pending-commands"
   12) "0"
   13) "last-ok-ping-reply"
   14) "340"
   15) "last-ping-reply"
   16) "340"
   17) "info-refresh"
   18) "2566"
   19) "master-link-down-time"
   20) "0"
   21) "master-link-status"
   22) "ok"
   23) "master-host"
   24) "localhost"
   25) "master-port"
   26) "6379"
   27) "slave-priority"
   28) "100"
```



- **SENTINEL get-master-addr-by-name mymaster** 查看指定 master-name 的地址

```
redis 127.0.0.1:26379> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6379"
```



断开的 sentinel 重新监控 master，其他 sentinel 收到事件信息

```
[6612] 18 Jul 16:32:53.512 * -dup-sentinel master mymaster 127.0.0.1 6379 #duplicate of 127.0.0.1:26379 or 6561ededb876035afd35f34bee089d06746327f1
[6612] 18 Jul 16:32:53.512 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6379
```



## 断开 master，触发 failover

**sentinel1 日志**

```
[6060] 18 Jul 16:33:50.440 # +sdown master mymaster 127.0.0.1 6379
[6060] 18 Jul 16:33:51.707 # +odown master mymaster 127.0.0.1 6379 #quorum 2/2
[6060] 18 Jul 16:33:51.707 # +failover-triggered master mymaster 127.0.0.1 6379
[6060] 18 Jul 16:33:51.707 # +failover-state-wait-start master mymaster 127.0.0.1 6379 #starting in 11992 milliseconds
[6060] 18 Jul 16:34:03.781 # +failover-state-select-slave master mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:03.881 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:03.883 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:03.984 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:03.986 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:03.987 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:04.085 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:05.091 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:06.165 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:06.285 # +failover-end master mymaster 127.0.0.1 6379
[6060] 18 Jul 16:34:06.285 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
[6060] 18 Jul 16:34:06.387 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
[6060] 18 Jul 16:34:06.577 * +sentinel sentinel 127.0.0.1:26380 127.0.0.1 26380 @ mymaster 127.0.0.1 6380
[6060] 18 Jul 16:34:11.602 * +sentinel sentinel 127.0.0.1:26381 127.0.0.1 26381 @ mymaster 127.0.0.1 6380
```



**sentinel2 日志**

```
[4620] 18 Jul 16:33:50.610 # +sdown master mymaster 127.0.0.1 6379
[4620] 18 Jul 16:33:50.810 # +odown master mymaster 127.0.0.1 6379 #quorum 2/2
[4620] 18 Jul 16:34:04.647 # +failover-detected master mymaster 127.0.0.1 6379
[4620] 18 Jul 16:34:05.200 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[4620] 18 Jul 16:34:06.245 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[4620] 18 Jul 16:34:06.347 # +failover-end master mymaster 127.0.0.1 6379
[4620] 18 Jul 16:34:06.350 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
[4620] 18 Jul 16:34:06.477 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
[4620] 18 Jul 16:34:06.507 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6380
[4620] 18 Jul 16:34:11.602 * +sentinel sentinel 127.0.0.1:26381 127.0.0.1 26381 @ mymaster 127.0.0.1 6380
```



**sentinel3 日志**

```
[6612] 18 Jul 16:34:09.927 # +sdown master mymaster 127.0.0.1 6379
[6612] 18 Jul 16:34:11.202 # +failover-detected master mymaster 127.0.0.1 6379
[6612] 18 Jul 16:34:11.302 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6612] 18 Jul 16:34:11.302 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6379
[6612] 18 Jul 16:34:11.402 # +failover-end master mymaster 127.0.0.1 6379
[6612] 18 Jul 16:34:11.402 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
[6612] 18 Jul 16:34:11.502 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
[6612] 18 Jul 16:34:11.622 * +sentinel sentinel 127.0.0.1:26379 127.0.0.1 26379 @ mymaster 127.0.0.1 6380
[6612] 18 Jul 16:34:11.702 * +sentinel sentinel 127.0.0.1:26380 127.0.0.1 26380 @ mymaster 127.0.0.1 6380
```



**注**：老版本的 redis 需要在 sentinel 的配置文件中设置 `sentinel can-failover mymaster yes`，sentinel 才有资格被选举为 leader 进行 failover。