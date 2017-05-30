---
categories: Java 面试
date: 2017-05-30 00:22
title: Java基础面试
---



###  Java 中 float 与 double 的区别

1. float是单精度浮点数，内存分配4个字节，占32位，有效小数位6-7位

   double是双精度浮点数，内存分配8个字节，占64位，有效小数位15位


2. java中默认声明的小数是double类型的，如double d=4.0

   如果声明： float x = 4.0则会报错，需要如下写法：float x = 4.0f或者float x = (float)4.0

   其中4.0f后面的f只是为了区别double，并不代表任何数字上的意义


3. 对编程人员而言，double 和 float 的区别是double精度高，但double消耗内存是float的两倍，且double的运算速度较float稍慢。



### 解释内存中的栈(stack)、堆(heap)和方法区(method area)的用法。

- `栈(stack)`：通常我们定义一个基本数据类型的变量，一个对象的引用，还有就是函数调用的现场保存都使用JVM中的栈空间；
- `堆(heap)`：而通过new关键字和构造器创建的对象则放在堆空间，**堆是垃圾收集器管理的主要区域**。由于现在的垃圾收集器都采用分代收集算法，所以堆空间还可以细分为新生代和老生代，再具体一点可以分为Eden、Survivor（又可分为From Survivor和To Survivor）、Tenured；
- `方法区(method area)`：方法区和堆都是各个线程共享的内存区域，用于存储已经被JVM加载的**类信息、常量、静态变量、JIT编译器编译后的代码等数据**；程序中的字面量（literal）如直接书写的100、"hello"和常量都是放在常量池中，**常量池是方法区的一部分**。
- 栈空间操作起来最快但是栈很小，通常大量的对象都是放在堆空间，栈和堆的大小都可以通过JVM的启动参数来进行调整，栈空间用光了会引发`StackOverflowError`，而堆和常量池空间不足则会引发`OutOfMemoryError`。

```
String str = new String("hello");
```

上面的语句中变量str放在栈上，用new创建出来的字符串对象放在堆上，而"hello"这个字面量是放在方法区的

> **补充1：**较新版本的Java（从Java 6的某个更新开始）中，由于JIT编译器的发展和"逃逸分析"技术的逐渐成熟，栈上分配、标量替换等优化技术使得对象一定分配在堆上这件事情已经变得不那么绝对了。
>
> **补充2**：运行时常量池相当于Class文件常量池具有动态性，Java语言并不要求常量一定只有编译期间才能产生，运行期间也可以将新的常量放入池中，String类的intern()方法就是这样的。



### String和StringBuilder、StringBuffer的区别

- Java平台提供了两种类型的字符串：`String`和`StringBuffer` / `StringBuilder`，它们可以储存和操作字符串。其中String是只读字符串，也就意味着String引用的字符串内容是不能被改变的。而StringBuffer/StringBuilder类表示的字符串对象可以直接进行修改。
- StringBuilder是Java 5中引入的，它和StringBuffer的方法完全相同，区别在于它是在单线程环境下使用的，因为它的所有方面都没有被synchronized修饰，因此它的效率也比StringBuffer要高。

> 补充：
>
> 1. String对象的intern方法会得到字符串对象在常量池中对应的版本的引用（如果常量池中有一个字符串与String对象的equals结果是true），如果常量池中没有对应的字符串，则该字符串将被添加到常量池中，然后返回常量池中字符串的引用；
> 2. 字符串的+操作其本质是创建了StringBuilder对象进行append操作，然后将拼接后的StringBuilder对象用toString方法处理成String对象，这一点可以用javap -c StringEqualTest.class命令获得class文件对应的JVM字节码指令就可以看出来。



### 重载（Overload）和重写（Override）的区别。重载的方法能否根据返回类型进行区分

- 方法的重载和重写都是实现多态的方式，区别在于前者实现的是`编译时的多态性`，而后者实现的是`运行时的多态性`。
- 重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；
- 重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。
- 重载对返回类型没有特殊的要求。
- ​

### 方法的重写规则

- `参数列表`必须完全与被重写方法的相同；

- `返回类型`必须完全与被重写方法的返回类型相同；

- `访问权限`不能比父类中被重写的方法的访问权限更低。例如：如果父类的一个方法被声明为public，那么在子类中重写该方法就不能声明为protected。（原因：运行期父类调用的方法被子类重写，如果访问权限更低，那就访问不到子类重写后的方法了）

- 父类的成员方法只能被它的子类重写。

- 声明为`final`的方法不能被重写。

- 声明为`static`的方法不能被重写，但是能够被再次声明。

  （原因：静态方法是属于类的方法，在编译期就使用编译出来的类型进行绑定了）

- 子类和父类在同一个包中，那么子类可以重写父类所有方法，除了声明为private和final的方法。

- 子类和父类不在同一个包中，那么子类只能够重写父类的声明为public和protected的非final方法。

- 重写的方法不能抛出新的`强制性异常`，或者比被重写方法声明的更广泛的强制性异常，反之则可以。重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。

  （原因：父类在方法调用的过程中捕获并处理了强制性异常，如果子类重写的方法抛出了新的强制性异常，则父类无法捕获。）

- 构造方法不能被重写。

- 如果不能继承一个方法，则不能重写这个方法。（访问权限问题）




### 描述一下JVM加载class文件的原理机制

- JVM中类的装载是由类加载器（ClassLoader）和它的子类来实现的，Java中的类加载器是一个重要的Java运行时系统组件，它负责在运行时查找和装入类文件中的类。 
- 由于Java的跨平台性，经过编译的Java源程序并不是一个可执行程序，而是一个或多个类文件。当Java程序需要使用某个类时，**JVM会确保这个类已经被加载、连接（验证、准备和解析）和初始化。**
- `加载`：类的加载是指把类的.class文件中的数据读入到内存中，通常是创建一个字节数组读入.class文件，然后产生与所加载类对应的Class对象。加载完成后，Class对象还不完整，所以此时的类还不可用。
- `链接`：当类被加载后就进入连接阶段，这一阶段包括验证、准备（为静态变量分配内存并设置默认的初始值）和解析（将符号引用替换为直接引用）三个步骤。
- `初始化`：最后JVM对类进行初始化，包括：
  - 1)如果类存在直接的父类并且这个类还没有被初始化，那么就先初始化父类；
  - 2)如果类中存在初始化语句，就依次执行这些初始化语句。



类的加载是由类加载器完成的，类加载器包括：`根加载器（BootStrap）`、`扩展加载器（Extension）`、`系统加载器（System）`和`用户自定义类加载器`（java.lang.ClassLoader的子类）。从Java 2（JDK 1.2）开始，**类加载过程采取了父亲委托机制（PDM）**。PDM更好的保证了Java平台的安全性，在该机制中，JVM自带的Bootstrap是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。JVM不会向Java程序提供对Bootstrap的引用。

- Bootstrap：一般用本地代码实现，负责加载JVM基础核心类库（rt.jar）；
- Extension：从java.ext.dirs系统属性所指定的目录中加载类库，它的父加载器是Bootstrap；
- System：又叫应用类加载器，其父类是Extension。它是应用最广泛的类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中记载类，是用户自定义加载器的默认父加载器。




### 抽象类（abstract class）和接口（interface）有什么异同

**相同**：

- 抽象类和接口都**不能够实例化**，但可以定义抽象类和接口类型的引用。
- 一个类如果继承了某个抽象类或者实现了某个接口都需要对其中的抽象方法全部进行实现，否则该类仍然需要被声明为抽象类。


**不同**：

- 接口比抽象类更加抽象，因为**抽象类中可以定义构造器**，可以有抽象方法和具体方法，而接口中不能定义构造器而且其中的方法全部都是抽象方法。
- 抽象类中的成员可以是private、default、protected、public的，而**接口中的成员全都是public**的。
- 抽象类中可以定义成员变量，而**接口中定义的成员变量实际上都是常量**。有抽象方法的类必须被声明为抽象类，而抽象类未必要有抽象方法。



**补充：为何“接口中定义的成员变量实际上都是常量”？**

> - 首先接口是一种高度抽象的"模版"，,而接口中的属性也就是’模版’的成员，就应当是所有实现"模版"的实现类的共有特性，所以它是public static的 ,是所有实现类共有的 .**假如可以是非static的话，因一个类可以继承多个接口，出现重名的变量，如何区分呢**？
> - 其次,接口中如果可能定义非final的变量的话，而方法又都是abstract的，这就自相矛盾了，有可变成员变量但对应的方法却无法操作这些变量，**虽然可以直接修改这些静态成员变量的值，但所有实现类对应的值都被修改了**，这跟抽象类有何区别? 接口是一种更高层面的抽象，是一种规范、功能定义的声明，所有可变的东西都应该归属到实现类中，这样接口才能起到标准化、规范化的作用。所以接口中的属性必然是final的。
> - 最后，接口只是对事物的属性和行为更高层次的抽象 。对修改关闭，对扩展（不同的实现implements）开放，接口是对开闭原则（[Open-Closed Principle](http://www.cnblogs.com/wxx/archive/2005/07/07/187582.html) ）的一种体现。



### 如何实现对象克隆？

答：有两种方式： 
  1). 实现Cloneable接口并重写Object类中的clone()方法； 
  2). 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆，代码如下：

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class MyUtil {

    private MyUtil() {
        throw new AssertionError();
    }

    @SuppressWarnings("unchecked")
    public static <T extends Serializable> T clone(T obj) throws Exception {
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bout);
        oos.writeObject(obj);

        ByteArrayInputStream bin = new ByteArrayInputStream(bout.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bin);
        return (T) ois.readObject();

        // 说明：调用ByteArrayInputStream或ByteArrayOutputStream对象的close方法没有任何意义
        // 这两个基于内存的流只要垃圾回收器清理对象就能够释放资源，这一点不同于对外部资源（如文件流）的释放
    }
}
```

**注意：**基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是好过把问题留到运行时。

### 一个".java"源文件中是否可以包含多个类（不是内部类）？有什么限制？

答：可以，但一个源文件中最多只能有一个公开类（public class）而且文件名必须和公开类的类名完全保持一致。



### Anonymous Inner Class(匿名内部类)是否可以继承其它类？是否可以实现接口？

答：可以继承其他类或实现其他接口，在Swing编程和Android开发中常用此方式来实现事件监听和回调。



### 内部类可以引用它的包含类（外部类）的成员吗？有没有什么限制？

答：一个内部类对象可以访问创建它的外部类对象的成员，包括私有成员。



### Java 中的final关键字有哪些用法？

- (1)修饰类：表示该类不能被继承；
- (2)修饰方法：表示方法不能被重写；
- (3)修饰变量：表示变量只能一次赋值以后值不能被修改（常量）。



### 子类继承父类，静态代码块，非晶态代码块和构造器的执行顺序

- 当new一个子类的对象时，先加载父类，执行父类的静态代码块（只加载时执行一次），然后执行子类的静态代码块（只加载时执行一次）
- 接着执行父类的非静态代码块（每次new一个对象时执行一次），然后执行子类的非静态代码块（每次new一个对象时执行一次）
- 最后执行父类的构造器，然后执行子类的构造器。



**40、怎样将GB2312编码的字符串转换为ISO-8859-1编码的字符串？** 
答：代码如下所示：

```java
String s1 = "你好";
String s2 = new String(s1.getBytes("GB2312"), "ISO-8859-1");
```



### 日期和时间

**- 如何取得年月日、小时分钟秒？** 
**- 如何取得从1970年1月1日0时0分0秒到现在的毫秒数？** 
**- 如何取得某月的最后一天？** 
**- 如何格式化日期？** 
答： 
问题1：创建`java.util.Calendar` 实例，调用其get()方法传入不同的参数即可获得参数所对应的值。Java 8中可以使用java.time.LocalDateTimel来获取，代码如下所示。

```java
public class DateTimeTest {
    public static void main(String[] args) {
        Calendar cal = Calendar.getInstance();
        System.out.println(cal.get(Calendar.YEAR));
        System.out.println(cal.get(Calendar.MONTH));    // 0 - 11
        System.out.println(cal.get(Calendar.DATE));
        System.out.println(cal.get(Calendar.HOUR_OF_DAY));
        System.out.println(cal.get(Calendar.MINUTE));
        System.out.println(cal.get(Calendar.SECOND));

        // Java 8
        LocalDateTime dt = LocalDateTime.now();
        System.out.println(dt.getYear());
        System.out.println(dt.getMonthValue());     // 1 - 12
        System.out.println(dt.getDayOfMonth());
        System.out.println(dt.getHour());
        System.out.println(dt.getMinute());
        System.out.println(dt.getSecond());
    }
}
```

问题2：以下方法均可获得该毫秒数。

```java
Calendar.getInstance().getTimeInMillis();
System.currentTimeMillis();
Clock.systemDefaultZone().millis(); // Java 8
```

问题3：代码如下所示。

```java
Calendar time = Calendar.getInstance();
time.getActualMaximum(Calendar.DAY_OF_MONTH);
```

问题4：利用java.text.DataFormat 的子类（如*SimpleDateFormat*类）中的format(Date)方法可将日期格式化。Java 8中可以用java.time.format.DateTimeFormatter来格式化时间日期，代码如下所示。

```java
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.Date;

class DateFormatTest {

    public static void main(String[] args) {
        SimpleDateFormat oldFormatter = new SimpleDateFormat("yyyy/MM/dd");
        Date date1 = new Date();
        System.out.println(oldFormatter.format(date1));

        // Java 8
        DateTimeFormatter newFormatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
        LocalDate date2 = LocalDate.now();
        System.out.println(date2.format(newFormatter));
    }
```



> 补充：Java的时间日期API一直以来都是被诟病的东西，为了解决这一问题，Java 8中引入了新的时间日期API，其中包括LocalDate、LocalTime、LocalDateTime、Clock、Instant等类，这些的类的设计都使用了不变模式，因此是线程安全的设计。如果不理解这些内容，可以参考我的另一篇文章[《关于Java并发编程的总结和思考》](http://blog.csdn.net/jackfrued/article/details/44499227)。



### Thread类的sleep()方法和对象的wait()方法都可以让线程暂停执行，它们有什么区别?

- `sleep()`方法（休眠）是线程类（Thread）的静态方法，调用此方法会让当前线程暂停执行指定的时间，将执行机会（CPU）让给其他线程，但是对象的**锁依然保持**，因此休眠时间结束后会自动恢复（线程回到**就绪状态**，请参考第66题中的线程状态转换图）。


- `wait()`是Object类的方法，调用对象的wait()方法导致当前线程**放弃对象的锁**（线程暂停执行），进入对象的**等待池（wait pool）**，只有调用对象的notify()方法（或notifyAll()方法）时才能唤醒等待池中的线程进入**等锁池（lock pool）**，如果线程重新获得对象的锁就可以进入**就绪状态**。



### 线程的sleep()方法和yield()方法有什么区别？

- sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行的机会；**yield()方法只会给相同优先级或更高优先级的线程以运行的机会；**
- 线程执行sleep()方法后转入阻塞（blocked）状态，而执行**yield()方法后转入就绪（ready）状态**； 
- sleep()方法声明抛出InterruptedException，而yield()方法没有声明任何异常； 
  - sleep()方法比yield()方法（跟操作系统CPU调度相关）具有更好的可移植性。



### 编写多线程程序有几种实现方式？

答：Java 5以前实现多线程有两种实现方法：一种是继承Thread类；另一种是实现Runnable接口。两种方式都要通过重写run()方法来定义线程的行为，推荐使用后者，因为Java中的继承是单继承，一个类有一个父类，如果继承了Thread类就无法再继承其他类了，显然使用Runnable接口更为灵活。

> 补充：Java 5以后创建线程还有第三种方式：实现Callable接口，该接口中的call方法可以在线程执行结束时产生一个返回值，代码如下所示：



### 什么是线程池（thread pool）

答：在面向对象编程中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或者其它更多资源。在Java中更是如此，虚拟机将试图跟踪每一个对象，以便能够在对象销毁后进行垃圾回收。所以提高服务程序效率的一个手段就是尽可能减少创建和销毁对象的次数，特别是一些很耗资源的对象创建和销毁，这就是”池化资源”技术产生的原因。线程池顾名思义就是事先创建若干个可执行的线程放入一个池（容器）中，需要的时候从池中获取线程不用自行创建，使用完毕不需要销毁线程而是放回池中，从而减少创建和销毁线程对象的开销。 
Java 5+中的Executor接口定义一个执行线程的工具。它的子类型即线程池接口是ExecutorService。要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，因此在工具类Executors面提供了一些静态工厂方法，生成一些常用的线程池，如下所示： 

- `newSingleThreadExecutor`：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。 

- `newFixedThreadPool`：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。 

- `newCachedThreadPool`：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。 

- `newScheduledThreadPool`：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。 

  ​

第60题的例子中演示了通过Executors工具类创建线程池并使用线程池执行线程的代码。如果希望在服务器上使用线程池，强烈建议使用`newFixedThreadPool`方法来创建线程池，这样能获得更好的性能。





**线程的基本状态以及状态之间的关系？** 

![java_thread](./java/thread.jpg)



### 简述synchronized 和java.util.concurrent.locks.Lock的异同？

Lock是Java 5以后引入的新的API，和关键字synchronized相比

**主要相同点**：Lock 能完成synchronized所实现的所有功能；

**主要不同点**：Lock有比synchronized更精确的线程语义和更好的性能，而且不强制性的要求一定要获得锁。synchronized会自动释放锁，而Lock一定要求程序员手工释放，并且最好在finally 块中释放（这是释放外部资源的最好的地方）



### Java中如何实现序列化，有什么意义？

答：序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化。可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间。序列化是为了解决对象流读写操作时可能引发的问题（如果不进行序列化可能会存在数据乱序的问题）。 
要实现序列化，需要让一个类实现Serializable接口，该接口是一个标识性接口，标注该类对象是可被序列化的，然后使用一个输出流来构造一个对象输出流并通过writeObject(Object)方法就可以将实现对象写出（即保存其状态）；如果需要反序列化则可以用一个输入流建立对象输入流，然后通过readObject方法从流中读取对象。序列化除了能够实现对象的持久化之外，还能够用于对象的深度克隆（可以参考第29题）。



### XML文档定义有几种形式？它们之间有何本质区别？解析XML文档有哪几种方式

- XML文档定义分为`DTD`和`Schema`两种形式，二者都是对XML语法的约束，其本质区别在于Schema本身也是一个XML文件，可以被XML解析器解析，而且可以为XML承载的数据定义类型，约束能力较之DTD更强大。
- 对XML的解析主要有`DOM`（文档对象模型，Document Object Model）、`SAX`（Simple API for XML）和`StAX`（Java 6中引入的新的解析XML的方式，Streaming API for XML），
  - `DOM`处理大型文件时其性能下降的非常厉害，这个问题是由DOM树结构占用的内存较多造成的，而且DOM解析方式必须在解析文件之前把**整个文档装入内存**，适合对XML的**随机访问**（典型的用空间换取时间的策略）；
  - `SAX`是事件驱动型的XML解析方式，它顺序读取XML文件，不需要一次全部装载整个文件。当遇到像文件开头，文档结束，或者标签开头与标签结束时，它会触发一个事件，用户通过事件回调代码来处理XML文件，适合对XML的**顺序访问**；
  - `StAX`把重点放在流上，实际上StAX与其他解析方式的本质区别就在于应用程序能够把XML作为一个事件流来处理。将XML作为一组事件来处理的想法并不新颖（SAX就是这样做的），但不同之处在于StAX允许应用程序代码把这些事件逐个拉出来，而不用提供在解析器方便时从解析器中接收事件的处理程序。



### 你在项目中哪些地方用到了XML？

答：XML的主要作用有两个方面：**数据交换和信息配置**。

- 在做数据交换时，XML将数据用标签组装成起来，然后压缩打包加密后通过网络传送给接收者，接收解密与解压缩后再从XML文件中还原相关信息进行处理，XML曾经是异构系统间交换数据的事实标准，但此项功能几乎已经被JSON（JavaScript Object Notation）取而代之。
- 当然，目前很多软件仍然使用XML来存储配置信息，我们在很多项目中通常也会将作为配置信息的硬代码写在XML文件中，Java的很多框架也是这么做的，而且这些框架都选择了[dom4j](http://www.dom4j.org/)作为处理XML的工具，因为Sun公司的官方API实在不怎么好用。





### 在进行数据库编程时，连接池有什么作用? 

答：由于创建连接和释放连接都有很大的开销（尤其是数据库服务器不在本地时，每次建立连接都需要进行TCP的三次握手，释放连接需要进行TCP四次握手，造成的开销是不可忽视的），为了提升系统访问数据库的性能，可以事先创建若干连接置于连接池中，需要时直接从连接池获取，使用结束时归还连接池而不必关闭连接，从而避免频繁创建和释放连接所造成的开销，这是典型的用空间换取时间的策略（浪费了空间存储连接，但节省了创建和释放连接的时间）。池化技术在Java开发中是很常见的，在使用线程时创建线程池的道理与此相同。基于Java的开源数据库连接池主要有：[C3P0](http://sourceforge.net/projects/c3p0/)、[Proxool](http://proxool.sourceforge.net/)、[DBCP](http://commons.apache.org/proper/commons-dbcp/)、[BoneCP](https://github.com/wwadge/bonecp)、[Druid](https://github.com/alibaba/druid)等。



**80、事务的ACID是指什么？** 
答： 

- `原子性(Atomic)`：事务中各项操作，要么全做要么全不做，任何一项操作的失败都会导致整个事务的失败； 
- `一致性(Consistent)`：事务结束后系统状态是一致的； 
- `隔离性(Isolated)`：并发执行的事务彼此无法看到对方的中间状态； 
- `持久性(Durable)`：事务完成后所做的改动都会被持久化，即使发生灾难性的失败。通过日志和同步备份可以在故障发生后重建数据。

> **补充：**
>
> 关于事务，在面试中被问到的概率是很高的，可以问的问题也是很多的。首先需要知道的是，只有存在并发数据访问时才需要事务。
>
> 当多个事务访问同一数据时，可能会存在5类问题，包括3类数据读取问题（脏读、不可重复读和幻读）和2类数据更新问题（第1类丢失更新和第2类丢失更新）。



### 获得一个类的类对象有哪些方式？

- 方法1：类型.class，例如：String.class 

- 方法2：对象.getClass()，例如："hello".getClass() 

- 方法3：Class.forName()，例如：Class.forName("java.lang.String")

  ​



**89、简述一下面向对象的"六原则一法则"。** 
答： 

- **单一职责原则**：一个类只做它该做的事情。（单一职责原则想表达的就是"高内聚"，写代码最终极的原则只有六个字"高内聚、低耦合"，所谓的高内聚就是一个代码模块只完成一项功能，在面向对象中，如果只让一个类完成它该做的事，而不涉及与它无关的领域就是践行了高内聚的原则，这个类就只有单一职责） 
- **开闭原则**：软件实体应当对扩展开放，对修改关闭。（在理想的状态下，当我们需要为一个软件系统增加新功能时，只需要从原来的系统派生出一些新类就可以，不需要修改原来的任何一行代码。要做到开闭有两个要点：①抽象是关键，一个系统中如果没有抽象类或接口系统就没有扩展点；②封装可变性，将系统中的各种可变因素封装到一个继承结构中，如果多个可变因素混杂在一起，系统将变得复杂而换乱，如果不清楚如何封装可变性，可以参考《设计模式精解》一书中对桥梁模式的讲解的章节。） 
- **依赖倒转原则**：面向接口编程。（该原则说得直白和具体一些就是声明方法的参数类型、方法的返回类型、变量的引用类型时，尽可能使用抽象类型而不用具体类型，因为抽象类型可以被它的任何一个子类型所替代，请参考下面的里氏替换原则。） 
- **里氏替换原则**：任何时候都可以用子类型替换掉父类型。（关于里氏替换原则的描述，Barbara Liskov女士的描述比这个要复杂得多，但简单的说就是能用父类型的地方就一定能使用子类型。里氏替换原则可以检查继承关系是否合理，如果一个继承关系违背了里氏替换原则，那么这个继承关系一定是错误的，需要对代码进行重构。需要注意的是：子类一定是增加父类的能力而不是减少父类的能力，因为子类比父类的能力更多，把能力多的对象当成能力少的对象来用当然没有任何问题。） 
- **接口隔离原则**：接口要小而专，绝不能大而全。（臃肿的接口是对接口的污染，既然接口表示能力，那么一个接口只应该描述一种能力，接口也应该是高度内聚的。Java中的接口代表能力、代表约定、代表角色，能否正确的使用接口一定是编程水平高低的重要标识。） 
- **合成聚合复用原则**：优先使用聚合或合成关系复用代码。（类与类之间简单的说有三种关系，Is-A关系、Has-A关系、Use-A关系，分别代表继承、关联和依赖。其中，关联关系根据其关联的强度又可以进一步划分为关联、聚合和合成，但说白了都是Has-A关系，合成聚合复用原则想表达的是优先考虑Has-A关系而不是Is-A关系复用代码，原因嘛可以自己从百度上找到一万个理由，需要说明的是，即使在Java的API中也有不少滥用继承的例子，例如Properties类继承了Hashtable类，Stack类继承了Vector类，这些继承明显就是错误的，更好的做法是在Properties类中放置一个Hashtable类型的成员并且将其键和值都设置为字符串来存储数据，而Stack类的设计也应该是在Stack类中放一个Vector对象来存储数据。**记住：任何时候都不要继承工具类，工具是可以拥有并可以使用的，而不是拿来继承的**。） 
- **迪米特法则**：迪米特法则又叫最少知识原则，一个对象应当对其他对象有尽可能少的了解。（迪米特法则简单的说就是如何做到"低耦合"，门面模式和调停者模式就是对迪米特法则的践行。



### 单例的几种写法（只列举线程安全）

```java
（饿汉）：
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}  
```

 这种方式基于classloder机制避免了多线程的同步问题，不过，instance在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用getInstance方法， 但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance显然没有达到lazy loading的效果。



```java
（静态内部类）：
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```

这种方式同样利用了classloder的机制来保证初始化instance时只有一个线程，它跟`饿汉模式`不同的是（很细微的差别）：饿汉模式只要Singleton类被装载了，那么instance就会被实例化（没有达到lazy loading效果），而这种方式是Singleton类被装载了，instance不一定被初始化。因为SingletonHolder类没有被主动使用，只有显示通过调用getInstance方法时，才会显示装载SingletonHolder类，从而实例化instance。想象一下，如果实例化instance很消耗资源，我想让他延迟加载，另外一方面，我不希望在Singleton类加载时就实例化，因为我不能确保Singleton类还可能在其他的地方被主动使用从而被加载，那么这个时候实例化instance显然是不合适的。这个时候，这种方式相比第三和第四种方式就显得很合理。



```java
（双重校验锁）：
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {
          // 双重校验锁的原因：
          // 尝试获得锁，但此时可能锁正被其他线程占用，当前线程挂起，
          // 当当前线程获得锁后，singleton可能已经被负值，所以需要双重校验锁
          if (singleton == null) {  
              singleton = new Singleton();  
          }  
        }  
    }  
    return singleton;  
    }  
}  
```

 这个是第二种方式的升级版，俗称双重检查锁定，详细介绍请查看：http://www.ibm.com/developerworks/cn/java/j-dcl.html
在JDK1.5之后，双重检查锁定才能够正常达到单例效果。



