---
categories: JVM
date: 2017-10-11 20:00
title: JDK 自带工具
---



## jps

```
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps
1376 QuorumPeerMain
1392 Jps
1364 QuorumPeerMain
1388 QuorumPeerMain

chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps -l
1376 org.apache.zookeeper.server.quorum.QuorumPeerMain
1393 sun.tools.jps.Jps
1364 org.apache.zookeeper.server.quorum.QuorumPeerMain
1388 org.apache.zookeeper.server.quorum.QuorumPeerMain

chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps -q
1376
1364
1388
1437

chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps -m
1376 QuorumPeerMain ./server/zk2/conf/zoo.cfg
1364 QuorumPeerMain ./server/zk1/conf/zoo.cfg
1388 QuorumPeerMain ./server/zk3/conf/zoo.cfg
1439 Jps -m

chenjianzhaodeMacBook-Pro:zookeeper cjz$ jps -v
1376 QuorumPeerMain -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false
1440 Jps -Dapplication.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home -Xms8m
1364 QuorumPeerMain -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false
1388 QuorumPeerMain -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false

```



## jstat

```

chenjianzhaodeMacBook-Pro:zookeeper cjz$ jstat -class 1376
Loaded  Bytes  Unloaded  Bytes     Time   
  1843  3400.8        0     0.0      13.43
  
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jstat -gc 1376
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
5120.0 5120.0  0.0   2048.0 33280.0  18830.1   87552.0      32.0    9984.0 9584.0 1280.0 1123.4      3    0.010   0      0.000    0.010

            
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jstat -gcnew 1376
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
5120.0 5120.0    0.0 2048.0  7  15 5120.0  33280.0  18830.1      3    0.010
```



## jinfo

```
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jinfo -flags  1376
Attaching to process ID 1376, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.131-b11
Non-default VM flags: -XX:CICompilerCount=3 -XX:InitialHeapSize=134217728 -XX:+ManagementServer -XX:MaxHeapSize=2147483648 -XX:MaxNewSize=715653120 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=44564480 -XX:OldSize=89653248 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseParallelGC 
Command line:  -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false
```



## jmap

```
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jmap -heap  1376
Attaching to process ID 1376, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.131-b11

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2147483648 (2048.0MB)
   NewSize                  = 44564480 (42.5MB)
   MaxNewSize               = 715653120 (682.5MB)
   OldSize                  = 89653248 (85.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 34078720 (32.5MB)
   used     = 20645536 (19.689117431640625MB)
   free     = 13433184 (12.810882568359375MB)
   60.58189978966346% used
From Space:
   capacity = 5242880 (5.0MB)
   used     = 2097184 (2.000030517578125MB)
   free     = 3145696 (2.999969482421875MB)
   40.0006103515625% used
To Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
PS Old Generation
   capacity = 89653248 (85.5MB)
   used     = 32768 (0.03125MB)
   free     = 89620480 (85.46875MB)
   0.03654970760233918% used

3600 interned Strings occupying 290208 bytes.

```



> $ jmap -histo  1376

```
chenjianzhaodeMacBook-Pro:zookeeper cjz$ jmap -histo  1376

 num     #instances         #bytes  class name
----------------------------------------------
   1:         53143        7764360  [B
   2:          1218        5205784  [I
   3:         37349        3780904  [C
   4:         26463         635112  java.lang.String
   5:         13820         442240  org.apache.zookeeper.server.quorum.QuorumPacket
   6:         10474         401088  [Ljava.lang.Object;
   7:         11666         373312  java.util.concurrent.locks.AbstractQueuedSynchronizer$Node
   8:          6889         330672  java.util.HashMap
   9:         10310         329920  java.util.HashMap$Node
  10:          3844         276768  org.apache.zookeeper.server.Request
  11:          8330         266560  java.io.DataInputStream
  12:          8320         266240  java.io.ByteArrayInputStream
  13:         10768         258432  java.util.ArrayList
  14:          2895         231656  [Ljava.util.HashMap$Node;
  15:          2037         230200  java.lang.Class
  16:          5053         202120  java.util.HashMap$KeyIterator
  17:          3342         187152  org.apache.zookeeper.server.DataTree$ProcessTxnResult
  18:          3843         184464  org.apache.zookeeper.txn.TxnHeader
  19:          5755         184160  java.util.ArrayList$Itr
  20:          3371         161808  java.nio.HeapByteBuffer
  21:          4995         119880  java.util.concurrent.LinkedBlockingQueue$Node
  22:          3348         107136  java.io.DataOutputStream
  23:          6622         105952  java.util.HashSet
  24:          2370          94800  java.util.ArrayList$SubList
  25:          2370          94800  java.util.ArrayList$SubList$1
  26:          3856          92544  java.util.LinkedList$Node

```

