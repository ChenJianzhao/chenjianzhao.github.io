---
categories: Spring
date: 2017-02-28 23:30
status: public
title: '安装启动 kafka 过程中遇到的异常'
---

## 安装启动``kafka``过程中遇到的异常
 
---
> 异常：Windows系统下启动 kafka 自带的 zookeeper 出现异常：
Couldnot find or load main class QuorumPeerMain
错误: 找不到或无法加载主类 org.apache.zookeeper.server.quorum.QuorumPeerMain

- 在 Kafka 的根目录下输入以下指令:
```
cd bin/windows
Then run Zookeper server:
zookeeper-server-start.bat ../../config/zookeeper.properties
Then run Kafka server:
kafka-server-start.bat ../../config/server.properties

```