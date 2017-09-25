### ByteBuf 使用模式

1. HEAP BUFFER(堆缓冲区)
2. DIRECT BUFFER(直接缓冲区)
3. COMPOSITE BUFFER(复合缓冲区)



#### HEAP BUFFER(堆缓冲区)

最常用的模式是 ByteBuf **将数据存储在 JVM 的堆空间**，这是通过将数据存储在数组的实现。堆缓冲区可以快速分配，当不使用时也可以快速释放。它还提供了直接访问支撑数组的方法，通过 ByteBuf.array() 来获取 byte[]数据。 

这种方法，正如清单5.1中所示的那样，是非常适合用来处理遗留数据的。

**Listing 5.1 Backing array**

```java
ByteBuf heapBuf = ...;
if (heapBuf.hasArray()) {               //1.检查 ByteBuf 是否有支持数组。
    byte[] array = heapBuf.array();		//2.如果有的话，得到引用数组。
    int offset = heapBuf.arrayOffset() + heapBuf.readerIndex();                												//3.计算第一字节的偏移量。
    int length = heapBuf.readableBytes();	 //4.获取可读的字节数
    handleArray(array, offset, length); 	 //5.使用数组，偏移量和长度作为调用方法的参数
}
```

注意：

- 访问非堆缓冲区 ByteBuf 的数组会导致UnsupportedOperationException， 可以使用 ByteBuf.hasArray()来检查是否支持访问数组。
- 这个用法与 JDK 的 ByteBuffer 类似



#### DIRECT BUFFER(直接缓冲区)

在 JDK1.4 中被引入 NIO 的ByteBuffer 类允许 JVM 通过本地方法调用分配内存，其目的是

- 通过免去中间交换的内存拷贝, 提升IO处理速度; 直接缓冲区的内容可以驻留在垃圾回收扫描的堆区以外。
- DirectBuffer 在 -XX:MaxDirectMemorySize=xxM大小限制下, 使用 Heap 之外的内存, GC对此”无能为力”,也就意味着规避了在高负载下频繁的GC过程对应用线程的中断影响.

这就解释了为什么“直接缓冲区”对于那些通过 socket 实现数据传输的应用来说，是一种非常理想的方式。如果你的数据是存放在堆中分配的缓冲区，那么实际上，在通过 socket 发送数据之前，**JVM 需要将先数据复制到直接缓冲区**。

但是直接缓冲区的缺点是在内存空间的分配和释放上比堆缓冲区**更复杂**，另外一个缺点是如果要将数据传递给遗留代码处理，因为数据不是在堆上，你可能不得不作出一个**副本**，如下：

**Listing 5.2 Direct buffer data access**

```java
ByteBuf directBuf = ...
if (!directBuf.hasArray()) {            
  //1.检查 ByteBuf 是不是由数组支持。如果不是，这是一个直接缓冲区。
    int length = directBuf.readableBytes();	//2.获取可读的字节数
    byte[] array = new byte[length];    	//3.分配一个新的数组来保存字节
    directBuf.getBytes(directBuf.readerIndex(), array); //4.字节复制到数组   
    handleArray(array, 0, length);  //5.将数组，偏移量和长度作为参数调用某些处理方法
}
```

显然，这比使用数组要多做一些工作。因此，如果你事前就知道容器里的数据将作为一个数组被访问，你可能更愿意使用堆内存。



