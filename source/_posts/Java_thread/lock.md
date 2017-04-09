---
categories:
  - 'Java Core'
  - 'Java 线程'
date: 2017-03-22 23:17
status: public
title: Java线程之Lock
---

## 重入锁
Java中的重入锁（即ReentrantLock）与Java内置锁一样，是一种排它锁。使用synchronized的地方一定可以用ReentrantLock代替。

重入锁需要显示请求获取锁，并显示释放锁。为了避免获得锁后，没有释放锁，而造成其它线程无法获得锁而造成死锁，一般建议将释放锁操作放在finally块里，如下所示。

```java
try{
  renentrantLock.lock();
  // 用户操作
} finally {
  renentrantLock.unlock();
}
```
如果重入锁已经被其它线程持有，则当前线程的lock操作会被阻塞。除了lock()方法之外，重入锁（或者说锁接口）还提供了其它获取锁的方法以实现不同的效果。

- ``lockInterruptibly()`` 该方法尝试获取锁，若获取成功立即返回；若获取不成功则阻塞等待。与lock方法不同的是，在阻塞期间，如果当前线程被**打断（interrupt）**则该方法抛出InterruptedException。该方法提供了一种解除死锁的途径。
- ``tryLock()`` 该方法试图获取锁，若该锁当前可用，则该方法立即获得锁并立即返回true；若锁当前不可用，则立即返回false。该方法**不会阻塞**，并提供给用户对于成功获利锁与获取锁失败进行不同操作的可能性。
- ``tryLock(long time, TimeUnit unit)`` 该方法试图获得锁，若该锁当前可用，则立即获得锁并立即返回true。若锁当前不可用，则等待相应的时间（由该方法的两个参数决定）:
 - 1）若该时间内锁可用，则获得锁，并返回true；
 - 2）若等待期间当前线程被打断，则抛出InterruptedException；
 - 3）若等待时间结束仍未获得锁，则返回false。
 
重入锁可定义为``公平锁``或``非公平锁``，默认实现为非公平锁。

``公平锁``是指多个线程获取锁被阻塞的情况下，锁变为可用时，最新申请锁的线程获得锁。可通过在重入锁（RenentrantLock）的构造方法中传入true构建公平锁，如Lock lock = new RenentrantLock(true)
``非公平锁``是指多个线程等待锁的情况下，锁变为可用状态时，哪个线程获得锁是随机的。synchonized相当于非公平锁。可通过在重入锁的构造方法中传入false或者使用无参构造方法构建非公平锁。

***

## 读写锁
锁可以保证原子性和可见性。而原子性更多是针对写操作而言。对于读多写少的场景，一个读操作无须阻塞其它读操作，只需要保证读和写或者写与写不同时发生即可。此时，如果使用重入锁（即排它锁），对性能影响较大。Java中的读写锁（ReadWriteLock）就是为这种读多写少的场景而创造的。
实际上，ReadWriteLock接口并非继承自Lock接口，ReentrantReadWriteLock也只实现了ReadWriteLock接口而未实现Lock接口。ReadLock和WriteLock，是ReentrantReadWriteLock类的静态内部类，它们实现了Lock接口。
一个**ReentrantReadWriteLock**实例包含一个**ReentrantReadWriteLock.ReadLock**实例和一个**ReentrantReadWriteLock.WriteLock**实例。通过**readLock()**和**writeLock()**方法可分别获得读锁实例和写锁实例，并通过Lock接口提供的获取锁方法获得对应的锁。
读写锁的锁定规则如下：
 - 获得读锁后，其它线程可获得读锁而不能获取写锁
 - 获得写锁后，其它线程既不能获得读锁也不能获得写锁

```java
package org.demo;

import java.util.Date;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;`

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
        new Thread(new Runnable() {
            @Override
            public void run() {
                readWriteLock.readLock().lock();
                try {
                    System.out.println(new Date() + "\tThread 1 started with read lock");
                    try {
                        Thread.sleep(2000);
                    } catch (Exception ex) {
                    }
                    System.out.println(new Date() + "\tThread 1 ended");
                } finally {
                    readWriteLock.readLock().unlock();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                readWriteLock.readLock().lock();
                try {
                    System.out.println(new Date() + "\tThread 2 started with read lock");
                    try {
                        Thread.sleep(2000);
                    } catch (Exception ex) {
                    }
                    System.out.println(new Date() + "\tThread 2 ended");
                } finally {
                    readWriteLock.readLock().unlock();
                }
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                Lock lock = readWriteLock.writeLock();
                lock.lock();
                try {
                    System.out.println(new Date() + "\tThread 3 started with write lock");
                    try {
                        Thread.sleep(2000);
                    } catch (Exception ex) {
                        ex.printStackTrace();
                    }
                    System.out.println(new Date() + "\tThread 3 ended");
                } finally {
                    lock.unlock();
                }
            }
        }).start();
    }
}
```
**输出结果**

![](~/00-05-35.jpg)

***
 
## 条件锁
``条件锁``只是一个帮助用户理解的概念，实际上并没有条件锁这种锁。对于每个重入锁，都可以通过``newCondition()``方法绑定若干个**条件对象**。

条件对象提供以下方法以实现不同的等待语义

- ``await()`` 调用该方法的前提是，当前线程已经**成功获得与该条件对象绑定的重入锁**，否则调用该方法时会抛出IllegalMonitorStateException。调用该方法外，当前线程会释放当前已经获得的锁（这一点与上文讲述的Java内置锁的wait方法一致），并且等待其它线程调用该条件对象的``signal()``或者``signalAll()``方法（这一点与Java内置锁wait后等待notify()或notifyAll()很像）。或者在等待期间，当前线程被打断，则wait()方法会抛出InterruptedException并清除当前线程的打断状态。
- ``await(long time, TimeUnit unit)``适用条件和行为与await()基本一致，唯一不同点在于，指定时间之内没有收到signal()或signalALL()信号或者线程中断时该方法会返回false;其它情况返回true。
- ``awaitNanos(long nanosTimeout)`` 调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出IllegalMonitorStateException。nanosTimeout指定该方法等待信号的的最大时间（单位为纳秒）。若指定时间内收到signal()或signalALL()则返回nanosTimeout减去已经等待的时间；若指定时间内有其它线程中断该线程，则抛出InterruptedException并清除当前线程的打断状态；若指定时间内未收到通知，则返回0或负数。
- ``awaitUninterruptibly()`` 调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出IllegalMonitorStateException。调用该方法后，结束等待的唯一方法是其它线程调用该条件对象的signal()或signalALL()方法。等待过程中如果当前线程被中断，该方法仍然会继续等待，同时保留该线程的中断状态。
- ``awaitUntil(Date deadline)`` 适用条件与行为与awaitNanos(long nanosTimeout)完全一样，唯一不同点在于它不是等待指定时间，而是等待由参数指定的某一时刻。

**signal()与signalAll()**
- ``signal()`` 若有一个或若干个线程在等待该条件变量，则该方法会唤醒其中的一个（具体哪一个，无法预测）。调用该方法的前提是当前线程持有该条件变量对应的锁，否则抛出IllegalMonitorStateException。
- ``signalALL()`` 若有一个或若干个线程在等待该条件变量，则该方法会唤醒所有等待。调用该方法的前提是当前线程持有该条件变量对应的锁，否则抛出IllegalMonitorStateException。

**调用条件等待的注意事项**

- 调用上述任意条件等待方法的**前提都是当前线程已经获得与该条件对象对应的重入锁**。
调用条件等待后，当前线程让出CPU资源。
- 上述等待方法结束后，方法返回的前提是它能重新获得与该条件对象对应的重入锁。如果无法获得锁，仍然会继续等待。这也是awaitNanos(long nanosTimeout)可能会返回负值的原因。
- 一旦条件等待方法返回，则当前线程肯定已经获得了对应的重入锁。
重入锁可以创建若干个条件对象，signal()和signalAll()方法**只能唤醒相同条件对象的等待**。
- **一个重入锁上可以生成多个条件变量，不同线程可以等待不同的条件，从而实现更加细粒度的的线程间通信**。