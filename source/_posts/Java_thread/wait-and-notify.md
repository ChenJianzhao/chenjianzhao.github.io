---
categories:
  - 'Java Core'
  - 'Java 线程'
date: 2017-03-12 14:51
status: public
title: 'Java线程之wait, notify 和 notifyAll'
---

参考并整合自 
[Java进阶（三）多线程开发关键技术](http://www.jasongj.com/java/multi_thread/)
[如何在 Java 中正确使用 wait, notify 和 notifyAll](http://developer.51cto.com/art/201508/487488.htm)

## 一、sleep和wait到底什么区别
***
1. 其实这个问题应该这么问——sleep和wait有什么相同点。因为这两个方法除了都能让当前线程暂停执行完，几乎没有其它相同点。

2. wait方法是Object类的方法，这意味着所有的Java类都可以调用该方法。sleep方法是Thread类的静态方法。

3. wait是在当前线程持有wait对象锁的情况下，**暂时放弃锁**，并让出CPU资源，并积极等待其它线程调用同一对象的notify或者notifyAll方法。注意，即使只有一个线程在等待，并且有其它线程调用了notify或者notifyAll方法，等待的线程只是被激活，但是**它必须得再次获得锁才能继续往下执行**。换言之，即使notify被调用，但只要锁没有被释放，原等待线程因为未获得锁仍然无法继续执行。



**此处通过生产者消费者问题来讲解如何使用 wait、notify 和 notifyAll 来实现线程间的通信。**
## 二、如何使用Wait
***
1. 调用哪个对象的wait 
正确的方法是对在多线程间共享的那个Object来使用wait。在生产者消费者问题中，这个共享的Object就是那个缓冲区队列。

2. 何时调用wait
wait方法需要释放锁，那么前提条件是它已经持有锁。所以wait和notify（或者notifyAll）方法都必须被包裹在synchronized语句块中（或者synchronized修饰的方法中），并且synchronized后锁的对象应该与调用wait方法的对象一样。否则抛出IllegalMonitorStateException
既然我们应该在synchronized的函数或是synchronized语句块锁住的对象里调用wait，那哪个对象应该被synchronized呢？答案是，那个 你希望上锁的对象就应该被synchronized，即那个在多个线程间被共享的对象。在生产者消费者问题中，应该被synchronized的就是那个缓冲区队列。

2. 永远在循环（loop）里调用 wait 和 notify，不是在 If 语句
if语句存在一些微妙的小问题，导致即使条件没被满足，你的线程你也有可能被错误地唤醒。所以如果你不在线程被唤 醒后再次使用while循环检查唤醒条件是否被满足，你的程序就有可能会出错——例如在缓冲区为满的时候生产者继续生成数据，或者缓冲区为空的时候消费者 开始小号数据。所以记住，永远在while循环而不是if语句中使用wait！

**基于以上认知，下面这个是使用wait和notify函数的规范代码模板：**
```java
// The standard idiom for calling the wait method in Java 
synchronized (sharedObject) { 
    while (condition) { 
        sharedObject.wait(); 
        // (Releases lock, and reacquires on wakeup) 
    } 
    // do action based upon condition e.g. take or put into queue 
} 
```
就像我之前说的一样，在while循环里使用wait的目的，是在线程被唤醒的前后都持续检查条件是否被满足。如果条件并未改变，wait被调用之前notify的唤醒通知就来了，那么这个线程就不会被真正地唤醒。

## 三、示例程序
***
以下是一段生产者和消费者利用 wait 和 notifyAll 方法通信的 Demo
```java
package org.demo;

import java.util.LinkedList;
import java.util.Queue;

/**
 * Created by jzchen on 2017/3/12 0012.
 */
public class ProducerComsumerDemo {



    public static void main(String[] args) {

        Queue<String> buffer = new LinkedList<>();
        int maxSize = 10;

        Producer producer = new Producer(buffer, maxSize, "Producer");
        Consumer consumer = new Consumer(buffer, maxSize, "Consumer");

        producer.start();
        consumer.start();

    }

    protected static class Producer extends Thread {

        private Queue<String> buffer = null;
        private int maxSize = 0;
        private String name = null;

        public Producer(Queue<String> buffer, int maxSize, String name) {
            this.buffer = buffer;
            this.maxSize = maxSize;
            this.name = name;
        }

        @Override
        public void run() {

            int product = 0;
            while (true) {
                synchronized (buffer) {

                    //队列已满时释放锁，等待消费
                    while (buffer.size() == maxSize) {
                        try {
                            buffer.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    //生产产品
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println("produced --> product " + product);
                    buffer.add(String.valueOf(product++));
                    buffer.notifyAll();
                }
            }
        }
    }

    protected static class Consumer extends Thread {

        private Queue<String> buffer = null;
        private int maxSize = 0;
        private String name = null;

        public Consumer(Queue<String> buffer, int maxSize, String name) {
            this.buffer = buffer;
            this.maxSize = maxSize;
            this.name = name;
        }

        @Override
        public void run() {

            while (true) {
                synchronized (buffer) {
                    //队列为空时释放锁，等待生产
                    while (buffer.isEmpty()) {
                        try {
                            buffer.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }

                    //消费产品
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    System.out.println("consume --> product " + buffer.poll());
                    buffer.notifyAll();
                }
            }
        }
    }
}
```

 **可能看到的结果**
 
![](~/微信截图_20170312214822.png)