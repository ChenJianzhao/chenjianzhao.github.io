学习Java GC机制的目的是为了实用，也就是为了在JVM出现问题时分析原因并解决之。通过学习，我觉得JVM监控与调优主要的着眼点在于如何配置、如何监控、如何优化3点上。



## 参数设置

在Java虚拟机的参数中，有3种表示方法（出自：http://www.cnblogs.com/wenfeng762/archive/2011/08/14/2137810.html），用“ps -ef |grep “java”命令，可以得到当前Java进程的所有启动参数和配置参数：

- 标准参数（-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
- 非标准参数（-X），默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
- 非Stable参数（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用（但是，这些参数往往是非常有用的）；

（额外的，-DpropertyName=“value”的形式定义了一些全局属性值，下面有介绍。）



## 标准参数

其实标准参数是用过Java的人都最熟悉的，就是你在运行java命令时后面加上的参数，如 java -version, java -jar 等，输入命令 `java -help` 或 `java -?` 就能获得当前机器所有java的标准参数列表。



### -client

设置jvm使用client模式，这是一般在pc机器上使用的模式，启动很快，但性能和内存管理效率并不高；多用于桌面应用；



### -server

使用server模式，启动速度虽然慢（比client模式慢10%左右），但是性能和内存管理效率很高，适用于服务器，用于生成环境、开发环境或测试环境的服务端；

如果没有指定-server或-client，JVM启动的时候会自动检测当前主机是否为服务器，如果是就以server模式启动，64位的JVM只有server模式，所以无法使用-client参数；

默认情况下，不同的启动模式，执行GC的方式有所区别：

| 启动模式   | 新生代GC方式 | 旧生代和持久代GC的方式 |
| ------ | ------- | ------------ |
| client | 串行      | 串行           |
| server | 并行      | 并发           |

### -classpath ( -cp)

JVM加载和搜索文件的目录路径，多个路径用;分隔。

注意，如果使用了-classpath，JVM就不会再搜索环境变量中定义的CLASSPATH路径。

**JVM搜索路径的顺序为：**

1. 先搜索JVM自带的jar或zip包（Bootstrat，搜索路径可以用 `System.getProperty(“sun.boot.class.path”)` 获得）；
2. 搜索 `JRE_HOME/lib/ext` 下的jar包（Extension，搜索路径可以用 `System.getProperty(“java.ext.dirs”)` 获得）；
3. 搜索用户自定义目录，顺序为：当前目录（.），CLASSPATH，-cp；（搜索路径用`System.getProperty(“java.class.path”)` 获得）



### -verbose

这是查询GC问题最常用的命令之一，具体参数如：

- -verbose:class

  输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。

- -verbose:gc

  输出每次GC的相关情况，后面会有更详细的介绍。

- -verbose:jni

  输出native方法调用的相关情况，一般用于诊断jni调用错误信息。



### -DpropertyName=value

定义系统的全局属性值，如配置文件地址等，如果value有空格，可以用-Dname=”space string”这样的形式来定义，用 `System.getProperty(“propertyName”)` 可以获得这些定义的属性值，在代码中也可以用`System.setProperty(“propertyName”,”value”)` 的形式来定义属性。



## 非标准参数

非标准参数，是在标准参数的基础上进行扩展的参数，输入`java -X`命令，能够获得当前JVM支持的所有非标准参数列表（你会发现，其实并不多哦）。

在不同类型的JVM中，采用的参数有所不同，
在讲解非标准参数时，请参考下面的图，对内存区域的大小有个形象的了解（下图出自：http://iamzhongyong.iteye.com/blog/1333100）：



### -Xmn

新生代内存大小的最大值，包括E区和两个S区的总和，使用方法如：-Xmn65535，-Xmn1024k，-Xmn512m，-Xmn1g (-Xms,-Xmx也是种写法)
-Xmn只能使用在JDK1.4或之后的版本中，（之前的1.3/1.4版本中，可使用-XX:NewSize设置年轻代大小，用-XX:MaxNewSize设置年轻代最大值）；
如果同时设置了-Xmn和-XX:NewSize，-XX:MaxNewSize，则谁设置在后面，谁就生效；如果同时设置了-XX:NewSize -XX:MaxNewSize与-XX:NewRatio则实际生效的值是：min(MaxNewSize,max(NewSize, heap/(NewRatio+1)))（看考：http://www.open-open.com/home/space.php?uid=71669&do=blog&id=8891）
在开发、测试环境，可以-XX:NewSize 和 -XX:MaxNewSize来设置新生代大小，但在线上生产环境，使用-Xmn一个即可（推荐），或者将-XX:NewSize 和 -XX:MaxNewSize设置为同一个值，这样能够防止在每次GC之后都要调整堆的大小（即：抖动，抖动会严重影响性能）

### -Xms

初始堆的大小，也是堆大小的最小值，默认值是总共的物理内存/64（且小于1G），默认情况下，当堆中可用内存小于40%(这个值可以用-XX: MinHeapFreeRatio 调整，如-X:MinHeapFreeRatio=30)时，堆内存会开始增加，一直增加到-Xmx的大小；

### -Xmx

堆的最大值，默认值是总共的物理内存/64（且小于1G），如果Xms和Xmx都不设置，则两者大小会相同，默认情况下，当堆中可用内存大于70%（这个值可以用-XX: MaxHeapFreeRatio 调整，如-X:MaxHeapFreeRatio=60）时，堆内存会开始减少，一直减小到-Xms的大小；
整个堆的大小=年轻代大小+年老代大小，堆的大小不包含持久代大小，如果增大了年轻代，年老代相应就会减小，官方默认的配置为年老代大小/年轻代大小=2/1左右（使用-XX:NewRatio可以设置-XX:NewRatio=5，表示年老代/年轻代=5/1）；
建议在开发测试环境可以用Xms和Xmx分别设置最小值最大值，但是在线上生产环境，Xms和Xmx设置的值必须一样，原因与年轻代一样——防止抖动；

### -Xss

这个参数用于设置每个线程的栈内存，默认1M，一般来说是不需要改的。除非代码不多，可以设置的小点，另外一个相似的参数是-XX:ThreadStackSize，这两个参数在1.6以前，都是谁设置在后面，谁就生效；1.6版本以后，-Xss设置在后面，则以-Xss为准，-XXThreadStackSize设置在后面，则主线程以-Xss为准，其它线程以-XX:ThreadStackSize为准。

### -Xrs

减少JVM对操作系统信号（OS Signals）的使用（JDK1.3.1之后才有效），当此参数被设置之后，jvm将不接收控制台的控制handler，以防止与在后台以服务形式运行的JVM冲突（这个用的比较少，参考：http://www.blogjava.net/midstr/archive/2008/09/21/230265.html）。

### -Xprof

跟踪正运行的程序，并将跟踪数据在标准输出输出；适合于开发环境调试。

### -Xnoclassgc

关闭针对class的gc功能；因为其阻止内存回收，所以可能会导致OutOfMemoryError错误，慎用；

### -Xincgc

开启增量gc（默认为关闭）；这有助于减少长时间GC时应用程序出现的停顿；但由于可能和应用程序并发执行，所以会降低CPU对应用的处理能力。

### -Xloggc:file

与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。
若与verbose命令同时出现在命令行中，则以-Xloggc为准。



| **参数名称**        | **含义**               | **默认值**         | 备注                                       |
| --------------- | -------------------- | --------------- | ---------------------------------------- |
| -Xms            | 初始堆大小                | 物理内存的1/64(<1GB) | 默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.(MinHeapFreeRatio参数可以调整) |
| -Xmx            | 最大堆大小                | 物理内存的1/4(<1GB)  | 默认空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制.(MaxHeapFreeRatio参数可以调整) |
| -Xmn            | 年轻代大小(1.4or lator)   |                 | **注意**：此处的大小是（eden+ 2 survivor space).整个堆大小=年轻代大小 + 年老代大小 + 持久代大小.增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8 |
| -XX:NewSize     | 设置年轻代大小(for 1.3/1.4) |                 |                                          |
| -XX:MaxNewSize  | 年轻代最大值(for 1.3/1.4)  |                 |                                          |
| -XX:PermSize    | 设置持久代(perm gen)初始值   | 物理内存的1/64       |                                          |
| -XX:MaxPermSize | 设置持久代最大值             | 物理内存的1/4        |                                          |
| -Xss            | 每个线程的堆栈大小            |                 | JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长）和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"”-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了。 |





原文：

[JVM监控与调优](http://www.importnew.com/21441.html)

