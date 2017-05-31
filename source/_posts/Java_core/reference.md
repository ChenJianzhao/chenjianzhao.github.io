---
categories:
  - Java Core
  - Java 进阶
date: 2017-06-01 00:00
title: Java的四种引用
---

## 引言

从JDK1.2版本开始，把对象的引用分为四种级别，从而使程序能更加灵活的控制对象的生命周期。这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

<!-- more -->



## 1. 强引用(StrongReference)

 强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。



## 2. 软引用(SoftReference)

如果一个对象只具有软引用，则内存空间足够，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。



## 3. 弱引用(WeakReference)

弱引用与软引用的区别在于：只具有弱引用的对象拥有**更短暂**的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此**不一定会很快发现**那些只具有弱引用的对象。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。  



## 4. 虚引用(PhantomReference)

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之 关联的引用队列中。  



## **总结：**

`WeakReference`与`SoftReference`都可以用来保存对象的实例引用，这两个类与垃圾回收有关。
`WeakReference`是弱引用，其中保存的对象实例可以被GC回收掉。这个类通常用于在某处保存对象引用，而又不干扰该对象被GC回收，通常用于Debug、内存监视工具等程序中。因为这类程序一般要求即要观察到对象，又不能影响该对象正常的GC过程。
最近在JDK的Proxy类的实现代码中也发现了Weakrefrence的应用，Proxy会把动态生成的Class实例暂存于一个由Weakrefrence构成的Map中作为Cache。

`SoftReference`是强引用，它保存的对象实例，除非JVM即将OutOfMemory，否则不会被GC回收。这个特性使得它特别适合设计对象Cache。对于Cache，我们希望被缓存的对象最好始终常驻内存，但是如果JVM内存吃紧，为了不发生OutOfMemoryError导致系统崩溃，必要的时候也允许JVM回收Cache的内存，待后续合适的时机再把数据重新Load到Cache中。这样可以系统设计得更具弹性。



## 5. 测试 弱引用

```java
import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;
import java.lang.ref.WeakReference;

/**
 * Created by cjz on 2017/5/31.
 */
public class TestReference {
    public static void main(String[] args) {

        ReferenceQueue<User> referenceQueue = new ReferenceQueue<>();
        WeakReference<User> user = new WeakReference<User>(new User("007","cjz"),referenceQueue);

        // 打印gc前 weakReference 中的对象
        System.out.println(user.get().number + "->" + user.get().name);

        // 手动gc
        System.gc();

        // 主线程休眠 1ms，给gc线程让出cpu
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        if(user.get() != null)
            // 若 weakReference 中的对象未被gc回收
            System.out.println(user.get().number + "->" + user.get().name);
        else{
            // 若 weakReference 中的对象被gc，该 weakReference 将被添加到 ReferenceQueue 中
            System.out.println("user has been gc");

            Reference<?> reference = null;
            while ( (reference = referenceQueue.poll())!=null) {
                System.out.println(reference + "->" + (reference.get() == null ? "null" : reference.get()));
            }
        }
    }

    public static class User {

        public User(String number,String name){
            this.number = number;
            this.name = name;
        }

        public String name;

        public String number;
    }
}
```

测试结果：

```
007->cjz
user has been gc
java.lang.ref.WeakReference@60e53b93->null

Process finished with exit code 0
```

