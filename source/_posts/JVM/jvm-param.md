---
categories: JVM
date: 2018-02-05 10:00
title: JVM 参数调优
---

学习Java GC机制的目的是为了实用，也就是为了在JVM出现问题时分析原因并解决之。通过学习，我觉得JVM监控与调优主要的着眼点在于如何配置、如何监控、如何优化3点上。



## 参数设置

在Java虚拟机的参数中，有3种表示方法（出自：[JVM参数说明](http://www.cnblogs.com/wenfeng762/archive/2011/08/14/2137810.html))

用 `ps -ef |grep "java"` 命令，可以得到当前Java进程的所有启动参数和配置参数：

- **标准参数（-）**，所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
- **非标准参数（-X）**，默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
- **非Stable参数（-XX）**，此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用（但是，这些参数往往是非常有用的）；

（额外的，**-DpropertyName=“value” **的形式定义了一些全局属性值，下面有介绍。）



## 标准参数

其实标准参数是用过Java的人都最熟悉的，就是你在运行java命令时后面加上的参数，如 java -version, java -jar 等，输入命令 `java -help` 或 `java -?` 就能获得当前机器所有java的标准参数列表。

### -client

设置jvm使用client模式，这是一般在pc机器上使用的模式，启动很快，但性能和内存管理效率并不高；多用于桌面应用；

### -srver

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

- **新生代内存大小的最大值**，包括E区和两个S区的总和，使用方法如：-Xmn65535，-Xmn1024k，-Xmn512m，-Xmn1g (-Xms,-Xmx也是种写法)

- -Xmn 只能使用在JDK1.4或之后的版本中，（之前的1.3/1.4版本中，可使用-XX:NewSize设置年轻代大小，用-XX:MaxNewSize设置年轻代最大值）；

- 如果同时设置了`-Xmn` 和 `-XX:NewSize，-XX:MaxNewSize`，**则谁设置在后面，谁就生效**；

- 如果同时设置了`-XX:NewSize, -XX:MaxNewSize` 与 `-XX:NewRatio` 则实际生效的值是：min(MaxNewSize,max(NewSize, heap/(NewRatio+1)))

  （参考：http://www.open-open.com/home/space.php?uid=71669&do=blog&id=8891）

- 在开发、测试环境，可以-XX:NewSize 和 -XX:MaxNewSize来设置新生代大小，但在线上生产环境，使用-Xmn一个即可（推荐），或者将-XX:NewSize 和 -XX:MaxNewSize设置为同一个值，**这样能够防止在每次GC之后都要调整堆的大小（即：抖动，抖动会严重影响性能）**

### -Xms

- **初始堆的大小**，也是堆大小的最小值，默认值是总共的物理内存/64（且小于1G），
- 默认情况下，当堆中可用内存小于40%(这个值可以用 `-XX: MinHeapFreeRatio` 调整，如-X:MinHeapFreeRatio=30)时，堆内存会开始增加，一直增加到-Xmx的大小；

### -Xmx

- **堆的最大值**，默认值是总共的物理内存/4（且小于1G），如果Xms和Xmx都不设置，则两者大小会相同，


- 默认情况下，当堆中可用内存大于70%（这个值可以用 `-XX: MaxHeapFreeRatio` 调整，如 -X:MaxHeapFreeRatio=60）时，堆内存会开始减少，一直减小到-Xms的大小；
- 整个堆的大小=年轻代大小+年老代大小，堆的大小不包含持久代大小，如果增大了年轻代，年老代相应就会减小，**官方默认的配置为年老代大小/年轻代大小=2/1左右**（使用-XX:NewRatio可以设置-XX:NewRatio=5，表示年老代/年轻代=5/1）；
- **注：**建议在开发测试环境可以用Xms和Xmx分别设置最小值最大值，但是在线上生产环境，Xms和Xmx设置的值必须一样，原因与年轻代一样——防止抖动；

### -Xss

- 这个参数用于设置每个线程的栈内存，默认1M，一般来说是不需要改的。除非代码不多，可以设置的小点，
- 另外一个相似的参数是 `-XX:ThreadStackSize`，这两个参数在1.6以前，都是谁设置在后面，谁就生效；1.6版本以后，-Xss设置在后面，则以-Xss为准，-XXThreadStackSize设置在后面，则主线程以-Xss为准，其它线程以-XX:ThreadStackSize为准。

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



## 非Stable参数

以-XX表示的非Stable参数，虽然在官方文档中是不确定的，不健壮的，各个公司的实现也各有不同，但往往非常实用，所以这部分参数对于GC非常重要。JVM（Hotspot）中主要的参数可以大致分为3类（参考http://blog.csdn.net/sfdev/article/details/2063928）：

- **性能参数**（ Performance Options）：用于JVM的性能调优和内存分配控制，如初始化内存大小的设置；
- **行为参数**（Behavioral Options）：用于改变JVM的基础行为，如GC的方式和算法的选择；
- **调试参数**（Debugging Options）：用于监控、打印、输出等jvm参数，用于显示jvm更加详细的信息；

比较详细的非Stable参数总结，请参考 [Java 6 JVM参数选项大全（中文版）](http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm)，
对于非Stable参数，使用方法有4种：

- `-XX:+<option>` 启用选项
- `-XX:-<option>` 不启用选项
- `-XX:<option>=<number>` 给选项设置一个数字类型值，可跟单位，例如 32k, 1024m, 2g
- `-XX:<option>=<string>` 给选项设置一个字符串值，例如-XX:HeapDumpPath=./dump.core

### 性能参数

首先介绍**性能参数**，性能参数往往用来定义内存分配的大小和比例，相比于行为参数和调试参数，一个比较明显的区别是性能参数后面往往跟的有数值，常用如下：

| 参数及其默认值                          | 描述                                       |
| -------------------------------- | ---------------------------------------- |
| **-XX:NewSize=2.125m**           | **新生代对象生成时占用内存的默认值**                     |
| **-XX:MaxNewSize=size**          | **新生成对象能占用内存的最大值**                       |
| **-XX:MaxPermSize=64m**          | **方法区所能占用的最大内存（非堆内存）**                   |
| -XX:PermSize=64m                 | 方法区分配的初始内存                               |
| -XX:MaxTenuringThreshold=15      | 对象在新生代存活区切换的次数（坚持过MinorGC的次数，每坚持过一次，该值就增加1），大于该值会进入老年代 |
| -XX:MaxHeapFreeRatio=70          | GC后java堆中空闲量占的最大比例，大于该值，则堆内存会减少          |
| -XX:MinHeapFreeRatio=40          | GC后java堆中空闲量占的最小比例，小于该值，则堆内存会增加          |
| -XX:NewRatio=2                   | 新生代内存容量与老生代内存容量的比例（XX:NewRatio=2 表示老年代/新生代=2） |
| -XX:ReservedCodeCacheSize= 32m   | 保留代码占用的内存容量                              |
| -XX:ThreadStackSize=512          | 设置线程栈大小，若为0则使用系统默认值（另见：-Xss 参数）          |
| -XX:LargePageSizeInBytes=4m      | 设置用于Java堆的大页面尺寸                          |
| -XX:PretenureSizeThreshold= size | 大于该值的对象直接晋升入老年代（这种对象少用为好）                |
| -XX:SurvivorRatio=8              | Eden区域Survivor区的容量比值，如默认值为8，代表Eden：Survivor1：Survivor2=8:1:1 |



### 行为参数

常用的行为参数，主要用来选择使用什么样的垃圾收集器组合，以及控制运行过程中的GC策略等：

| 参数及其默认值                            | 描述                                       |
| ---------------------------------- | ---------------------------------------- |
| **-XX:+UseSerialGC**               | **启用串行GC，即采用Serial+Serial Old模式**        |
| **-XX:+UseParallelGC**             | **启用并行GC，即采用Parallel Scavenge+Serial Old收集器组合（-Server模式下的默认组合）** |
| -XX:GCTimeRatio=99                 | 设置用户执行时间占总时间的比例（默认值99，即1%的时间用于GC）（这个参数只对Parallel Scavenge有效） |
| -XX:MaxGCPauseMillis=time          | 设置GC的最大停顿时间（这个参数只对Parallel Scavenge有效）   |
| -XX:-UseAdaptiveSizePolicy         | 动态调整 Java 内存各个区域的大小和进入老年代的年龄（这个参数只对Parallel Scavenge有效） |
| **-XX:+UseParNewGC**               | **使用ParNew+Serial Old收集器组合**             |
| -XX:ParallelGCThreads              | 设置执行内存回收的线程数，在+UseParNewGC的情况下使用         |
| **-XX:+UseParallelOldGC**          | 使用Parallel Scavenge +Parallel Old组合收集器   |
| **-XX:+UseConcMarkSweepGC**        | **使用ParNew+CMS+Serial Old组合并发收集，优先使用ParNew+CMS，当用户线程内存不足时，采用备用方案Serial Old收集。** |
| -XX:-DisableExplicitGC             | 禁止调用System.gc()；但jvm的gc仍然有效              |
| -XX:+ScavengeBeforeFullGC          | 新生代GC优先于Full GC执行                        |
| -XX:+UseCMSCompactAtFullCollection | 设置CMS收集器在完成垃圾收集后是否进行一次内存碎片整理             |
| -XX:CMSFullGCsBeforeCompation      | 设置CMS收集器在完成若干次垃圾收集后再进行一次内存碎片整理           |



### 调试参数

常用的调试参数，主要用于监控和打印GC的信息：

| 参数及其默认值                                  | 描述                                       |
| ---------------------------------------- | ---------------------------------------- |
| -XX:-CITime                              | 打印消耗在JIT编译的时间                            |
| -XX:ErrorFile=./hs_err_pid\<pid>.log     | 保存错误日志或者数据到文件中                           |
| -XX:-ExtendedDTraceProbes                | 开启solaris特有的dtrace探针                     |
| **-XX:HeapDumpPath=./java_pid\<pid>.hprof** | **指定导出堆信息时的路径或文件名**                      |
| **-XX:+HeapDumpOnOutOfMemoryError**      | **当首次遭遇OOM时导出此时堆中相关信息**                  |
| -XX:OnError=”\<cmd args>;\<cmd args>”    | 出现致命ERROR之后运行自定义命令                       |
| -XX:OnOutOfMemoryError=”\<cmd args>;\<cmd args>” | 当首次遭遇OOM时执行自定义命令                         |
| -XX:-PrintClassHistogram                 | 遇到Ctrl-Break后打印类实例的柱状信息，与jmap -histo功能相同 |
| **-XX:-PrintConcurrentLocks**            | **遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同** |
| -XX:-PrintCommandLineFlags               | 打印在命令行中出现过的标记                            |
| -XX:-PrintCompilation                    | 当一个方法被编译时打印相关信息                          |
| -XX:-PrintGC                             | 每次GC时打印相关信息                              |
| -XX:-PrintGC Details                     | 每次GC时打印详细信息                              |
| -XX:-PrintGCTimeStamps                   | 打印每次GC的时间戳                               |
| -XX:-TraceClassLoading                   | 跟踪类的加载信息                                 |
| -XX:-TraceClassLoadingPreorder           | 跟踪被引用到的所有类的加载信息                          |
| -XX:-TraceClassResolution                | 跟踪常量池                                    |
| -XX:-TraceClassUnloading                 | 跟踪类的卸载信息                                 |
| -XX:-TraceLoaderConstraints              | 跟踪类加载器约束的相关信息                            |

再次声明，上面的三种参数，主要参考了博客：http://blog.csdn.net/sfdev/article/details/2063928和http://kenwublog.com/docs/java6-jvm-options-chinese-edition.htm，后一个比较全面，有兴趣的可以仔细研读。
这些参数将为我们进行GC的监控与调优提供很大助力，是我们进行GC相关操作的重要工具。



## 面试题：

- 老年代反复 GC 是什么原因？（以下答案仅代表个人看法）

  1. 老年代的内存空间太小，可以调大参数 `-XX:NewRatio` 的值，增加老年代比例。

  2. 太多对象进入老年代，可以调大参数 `-XX:MaxTenuringThreshold` 的值，增大新生代对象进入老年代的门槛。

  3. 太多大对象直接进入老年代，可以调大参数 `-XX:PreTenureSizeThreshold` 的值，增大大对象直接进入老年代的门槛。

     ​

  ​

原文：（本文摘抄部分并略有修改）

[JVM监控与调优](http://www.importnew.com/21441.html)

参考：

http://wiki.jikexueyuan.com/project/jvm-parameter/throughput-collector.html