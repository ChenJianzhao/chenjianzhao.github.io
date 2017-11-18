---
categories:
  - Java Core
  - Java 集合
date: 2017-06-01 10:50
title: WeakHashMap 简介和示例
---



## WeakHashMap简介

- WeakHashMap 继承于AbstractMap，实现了Map接口。
- 和[HashMap](http://www.cnblogs.com/skywang12345/p/3310835.html)一样，WeakHashMap 也是一个**散列表**，它存储的内容也是**键值对(key-value)映射**，而且**键和值都可以是null**。
- 不过WeakHashMap的**键是“弱键”**。在 WeakHashMap 中，当某个键不再正常使用时，会被从WeakHashMap中被自动移除。更精确地说，对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的丢弃，这就使该键成为可终止的，被终止，然后被回收。某个键被终止时，它对应的键值对也就从映射中有效地移除了。
- 这个“弱键”的原理呢？大致上就是，**通过WeakReference和ReferenceQueue实现的**。 WeakHashMap的key是“弱键”，即是WeakReference类型的；ReferenceQueue是一个队列，它会保存被GC回收的“弱键”。

<!-- more -->

- 实现步骤是：

  1. 新建WeakHashMap，将“**键值对**”添加到WeakHashMap中。

  ​       实际上，WeakHashMap是通过数组table保存Entry(键值对)；每一个Entry实际上是一个单向链表，即Entry是键值对链表。

  2. 当**某“弱键”不再被其它对象引用**，并**被GC回收**时。在GC回收该“弱键”时，**这个“弱键”也同时会被添加到ReferenceQueue(queue)队列**中。
  3. 当下一次我们需要操作WeakHashMap时，会先同步table和queue。table中保存了全部的键值对，而queue中保存被GC回收的键值对；同步它们，就是**删除table中被GC回收的键值对**。

     这就是“弱键”如何被自动从WeakHashMap中删除的步骤了。

- 和HashMap一样，WeakHashMap是不同步的。可以使用 Collections.synchronizedMap 方法来构造同步的 WeakHashMap。



> 有关弱引用请参考 [Java的四种引用](../../Java_core/reference)

<!-- more -->

</br>

## WeakHashMap 使用示例

```java
import java.util.Map;
import java.util.WeakHashMap;

/**
 * Created by pc on 2017/6/1.
 */
public class TestWeakHashMap {

    public static void main(String[] args) {

        Map<User,String> weakHashMap = new WeakHashMap<>();

        User a = new User("a","a");
        User b = new User("b","b");

        weakHashMap.put(a,"userA");
        weakHashMap.put(b,"userB");

        a = null;		//将 UserA 的强引用置空
        // b = null;	//保留 UserB 的强引用

        System.gc();

        try {
          	// 为 JVM gc 线程留出时间
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        for(User user : weakHashMap.keySet()) {
            System.out.println(user);
        }
    }



    public static class User{

        String number = null;
        String name = null;

        public User(String number, String name) {
            this.number = number;
            this.name = name;
        }

        @Override
        public String toString() {
            return this.number + ":" + name;
        }
    }
}

```



运行结果

```
b:b
Process finished with exit code 0
```

</br>

## WeakHashMap 关键源码解析

- 首先看一下 `WeakHashMap` 中  `Entry` 的定义：
  - `Entry` 继承了 `WeakReference` ，将 `key` 和一个 `ReferenceQueue` 作为参数传入父类的构造器。
  - 从传入的参数看出 `WeakHashMap` 中的 `key` 是弱引用，在 JVM gc 时会被回收，并把`Entry` 本身加入到 `ReferenceQueue`  中。

```java
/**
 * Reference queue for cleared WeakEntries
 */
 private final ReferenceQueue<Object> queue = new ReferenceQueue<>();

...
...
...
  
/**
 * The entries in this hash table extend WeakReference, using its main ref
 * field as the key.
 */
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
  V value;
  final int hash;
  Entry<K,V> next;

  /**
         * Creates new entry.
         */
  Entry(Object key, V value,
        ReferenceQueue<Object> queue,
        int hash, Entry<K,V> next) {
    super(key, queue);
    this.value = value;
    this.hash  = hash;
    this.next  = next;
  }
  
  ...
  ...
  ...
}
```



- 当 `WeakHashMap` 的 key 被 gc 之后，`WeakHashMap` 通过 `expungeStaleEntries` 方法删除被回收的条目

```java
/**
 * Expunges stale entries from the table.
 */
private void expungeStaleEntries() {
  for (Object x; (x = queue.poll()) != null; ) {
    synchronized (queue) {
      @SuppressWarnings("unchecked")
      Entry<K,V> e = (Entry<K,V>) x;
      int i = indexFor(e.hash, table.length);

      Entry<K,V> prev = table[i];
      Entry<K,V> p = prev;
      while (p != null) {
        Entry<K,V> next = p.next;
        if (p == e) {			// 【1】当前项目等于待移除项目
          if (prev == e)		// 【1.1】链头等于待移除项目，链头 = e.next，移除 e
            table[i] = next;
          else
            prev.next = next;	 // 【1.2】e.prev.next = e.next，移除 e
          // Must not null out e.next;
          // stale entries may be in use by a HashIterator
          e.value = null; // Help GC
          size--;
          break;
        }
        prev = p;			// 【2】当前项目不等于待移除项目，指针下移
        p = next;
      }
    }
  }
}
```



参考：

[Java 集合系列13之 WeakHashMap详细介绍(源码解析)和使用示例](http://www.cnblogs.com/skywang12345/p/3311092.html)

[Java WeakHashMap 源码解析](http://liujiacai.net/blog/2015/09/27/java-weakhashmap/#WeakHashMap-expungeStaleEntries)