---
categories:
  - Java Core
  - Java NIO
title: Java NIO（十一）之 Pipe
---

##  Pipe

Java NIO 管道是2个线程之间的单向数据连接。 Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

这里是Pipe原理的图示：



## 1. 创建管道

通过`Pipe.open()`方法打开管道。例如：

```java
Pipe pipe = Pipe.open();
```



## 2. 向管道写数据

要向管道写数据，需要访问sink通道。像这样：

```java
Pipe.SinkChannel sinkChannel = pipe.sink();
```

通过调用SinkChannel的`write()`方法，将数据写入`SinkChannel`,像这样：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
 
buf.flip();
 
while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```



## 3. 从管道读取数据

从读取管道的数据，需要访问source通道，像这样：

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```

调用source通道的`read()`方法来读取数据，像这样：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
 int bytesRead = sourceChannel.read(buf);
```

`read()`方法返回的int值会告诉我们多少字节被读进了缓冲区。