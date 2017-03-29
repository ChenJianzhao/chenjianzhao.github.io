---
date: 2016-12-09 17:30
status: public
title: 设计模式（Java实现）
---

##设计模式的分类
***
总体来说设计模式分为三大类：
1. **创建型模式**，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。
2. **结构型模式**，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
3. **行为型模式**，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。
其实还有两类：**并发型模式和线程池模式**。















##9、外观模式（Facade）
实现描述：使用一个facade类，将分散的几个类连接起来，消除零散类之间的引用。

- 外观模式是为了解决类与类之家的依赖关系的，像spring一样，可以将类和类之间的关系配置到配置文件中，而外观模式就是将他们的关系放在一个Facade类中，降低了类类之间的耦合度，该模式中没有涉及到接口，看下类图：（我们以一个计算机的启动过程为例）
- 如果我们没有Computer类，那么，CPU、Memory、Disk他们之间将会相互持有实例，产生关系，这样会造成严重的依赖，修改一个类，可能会带来其他类的修改，这不是我们想要看到的，有了Computer类，他们之间的关系被放在了Computer类里，这样就起到了解耦的作用，这，就是外观模式！

如果我们没有Computer类，那么，CPU、Memory、Disk他们之间将会相互持有实例，产生关系，这样会造成严重的依赖，修改一个类，可能会带来其他类的修改，这不是我们想要看到的，有了Computer类，他们之间的关系被放在了Computer类里，这样就起到了解耦的作用，这，就是外观模式！

```java
public class CPU {  
      
    public void startup(){  
        System.out.println("cpu startup!");  
    }  
      
    public void shutdown(){  
        System.out.println("cpu shutdown!");  
    }  
}  

public class Memory {  
      
    public void startup(){  
        System.out.println("memory startup!");  
    }  
      
    public void shutdown(){  
        System.out.println("memory shutdown!");  
    }  
}  

public class Disk {  
      
    public void startup(){  
        System.out.println("disk startup!");  
    }  
      
    public void shutdown(){  
        System.out.println("disk shutdown!");  
    }  
}  

public class Computer {  
    private CPU cpu;  
    private Memory memory;  
    private Disk disk;  
      
    public Computer(){  
        cpu = new CPU();  
        memory = new Memory();  
        disk = new Disk();  
    }  
      
    public void startup(){  
        System.out.println("start the computer!");  
        cpu.startup();  
        memory.startup();  
        disk.startup();  
        System.out.println("start computer finished!");  
    }  
      
    public void shutdown(){  
        System.out.println("begin to close the computer!");  
        cpu.shutdown();  
        memory.shutdown();  
        disk.shutdown();  
        System.out.println("computer closed!");  
    }  
}  

//User类如下：
public class User {  
  
    public static void main(String[] args) {  
        Computer computer = new Computer();  
        computer.startup();  
        computer.shutdown();  
    }  
}  
```

## 10、桥接模式（Bridge）
实现描述：桥接模式就是把事物和其具体实现分开，使他们可以各自独立的变化。桥接的用意是：**将抽象化与实现化解耦，使得二者可以独立变化**

```java
public interface Sourceable {  
    public void method();  
}  

//分别定义两个实现类：
public class SourceSub1 implements Sourceable {  
  
    @Override  
    public void method() {  
        System.out.println("this is the first sub!");  
    }  
}  

public class SourceSub2 implements Sourceable {  
  
    @Override  
    public void method() {  
        System.out.println("this is the second sub!");  
    }  
}  

//定义一个桥，持有Sourceable的一个实例：
public abstract class Bridge {  
    private Sourceable source;  
  
    public void method(){  
        source.method();  
    }  
      
    public Sourceable getSource() {  
        return source;  
    }  
  
    public void setSource(Sourceable source) {  
        this.source = source;  
    }  
}  

public class MyBridge extends Bridge {  
    public void method(){  
        getSource().method();  
    }  
}  

//测试类：
public class BridgeTest {  
      
    public static void main(String[] args) {  
          
        Bridge bridge = new MyBridge();  
          
        /*调用第一个对象*/  
        Sourceable source1 = new SourceSub1();  
        bridge.setSource(source1);  
        bridge.method();  
          
        /*调用第二个对象*/  
        Sourceable source2 = new SourceSub2();  
        bridge.setSource(source2);  
        bridge.method();  
    }  
}  
```

## 11、组合模式（Composite）

组合模式有时又叫**部分-整体模式**在处理类似树形结构的问题时比较方便
```java
public class TreeNode {  
      
    private String name;  
    private TreeNode parent;  
    private Vector<TreeNode> children = new Vector<TreeNode>();  
      
    public TreeNode(String name){  
        this.name = name;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
  
    public TreeNode getParent() {  
        return parent;  
    }  
  
    public void setParent(TreeNode parent) {  
        this.parent = parent;  
    }  
      
    //添加孩子节点  
    public void add(TreeNode node){  
        children.add(node);  
    }  
      
    //删除孩子节点  
    public void remove(TreeNode node){  
        children.remove(node);  
    }  
      
    //取得孩子节点  
    public Enumeration<TreeNode> getChildren(){  
        return children.elements();  
    }  
}  

public class Tree {  
  
    TreeNode root = null;  
  
    public Tree(String name) {  
        root = new TreeNode(name);  
    }  
  
    public static void main(String[] args) {  
        Tree tree = new Tree("A");  
        TreeNode nodeB = new TreeNode("B");  
        TreeNode nodeC = new TreeNode("C");  
          
        nodeB.add(nodeC);  
        tree.root.add(nodeB);  
        System.out.println("build the tree finished!");  
    }  
}  
```

## 12、享元模式（Flyweight）
享元模式的主要目的是实现对象的共享，即共享池，当系统中对象多的时候可以减少内存的开销，通常与工厂模式一起使用。
一提到共享池，我们很容易联想到Java里面的JDBC连接池，想想每个连接的特点，我们不难总结出：适用于作共享的一些个对象，他们有一些共有的属性，就拿数据库连接池来说，url、driverClassName、username、password及dbname，这些属性对于每个连接来说都是一样的，所以就适合用享元模式来处理，建一个工厂类，将上述类似属性作为内部数据，其它的作为外部数据，在方法调用时，当做参数传进来，这样就节省了空间，减少了实例的数量。

看个例子：
实现描述：共享池中持有所有共享对象的引用，

```java
public class ConnectionPool {  
      
    private Vector<Connection> pool;  
      
    /*公有属性*/  
    private String url = "jdbc:mysql://localhost:3306/test";  
    private String username = "root";  
    private String password = "root";  
    private String driverClassName = "com.mysql.jdbc.Driver";  
  
    private int poolSize = 100;  
    private static ConnectionPool instance = null;  
    Connection conn = null;  
  
    /*构造方法，做一些初始化工作*/  
    private ConnectionPool() {  
        pool = new Vector<Connection>(poolSize);  
  
        for (int i = 0; i < poolSize; i++) {  
            try {  
                Class.forName(driverClassName);  
                conn = DriverManager.getConnection(url, username, password);  
                pool.add(conn);  
            } catch (ClassNotFoundException e) {  
                e.printStackTrace();  
            } catch (SQLException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
  
    /* 返回连接到连接池 */  
    public synchronized void release() {  
        pool.add(conn);  
    }  
  
    /* 返回连接池中的一个数据库连接 */  
    public synchronized Connection getConnection() {  
        if (pool.size() > 0) {  
            Connection conn = pool.get(0);  
            pool.remove(conn);  
            return conn;  
        } else {  
            return null;  
        }  
    }  
}  
```
---
## 十一种行为型模式

---
先来张图，看看这11中模式的关系：
第一类：通过父类与子类的关系进行实现。第二类：两个类之间。第三类：类的状态。第四类：通过中间类

## 13、策略模式（strategy）

策略模式定义了一系列算法，并将每个算法封装起来，使他们可以相互替换，且算法的变化不会影响到使用算法的客户。需要设计一个接口，为一系列实现类提供统一的方法，多个实现类实现该接口，设计一个抽象类（可有可无，属于辅助类），提供辅助函数。

实现描述：
定义一个接口，为一系列实现类提供统一的方法，多个实现类实现该接口，设计一个抽象类（可有可无，属于辅助类），提供辅助函数。实现类继承辅助抽象类，实现统一接口，分别提供不同的实现，供用户选择。

```java
    public int calculate(String exp);  
} 
 
//辅助类：
public abstract class AbstractCalculator {  
      
    public int[] split(String exp,String opt){  
        String array[] = exp.split(opt);  
        int arrayInt[] = new int[2];  
        arrayInt[0] = Integer.parseInt(array[0]);  
        arrayInt[1] = Integer.parseInt(array[1]);  
        return arrayInt;  
    }  
}  

//三个实现类：
public class Plus extends AbstractCalculator implements ICalculator {  
  
    @Override  
    public int calculate(String exp) {  
        int arrayInt[] = split(exp,"\\+");  
        return arrayInt[0]+arrayInt[1];  
    }  
}  

public class Minus extends AbstractCalculator implements ICalculator {  
  
    @Override  
    public int calculate(String exp) {  
        int arrayInt[] = split(exp,"-");  
        return arrayInt[0]-arrayInt[1];  
    }  
  
}  

public class Multiply extends AbstractCalculator implements ICalculator {  
  
    @Override  
    public int calculate(String exp) {  
        int arrayInt[] = split(exp,"\\*");  
        return arrayInt[0]*arrayInt[1];  
    }  
}  

//简单的测试类：
public class StrategyTest {  
  
    public static void main(String[] args) {  
        String exp = "2+8";  
        ICalculator cal = new Plus();  
        int result = cal.calculate(exp);  
        System.out.println(result);  
    }  
}  
```

> 注：**策略模式的决定权在用户**，系统本身提供不同算法的实现，新增或者删除算法，对各种算法做封装。因此，策略模式多用在算法决策系统中，外部用户只需要决定用哪个算法即可。

## 14、模板方法模式（Template Method）

解释一下模板方法模式，就是指：一个抽象类中，有一个主方法，再定义1...n个方法，可以是抽象的，也可以是实际的方法，主方法调用抽象方法；再定义一个子类，继承该抽象类，重写那1...n个抽象方法，子类通过调用抽象类的主方法，主方法调用抽象方法时会从子类逐级向上寻找具体实现，实现对子类的调用。

```java
public abstract class AbstractCalculator {  
      
    /*主方法，实现对本类其它方法的调用*/  
    public final int calculate(String exp,String opt){  
        int array[] = split(exp,opt);  
        return calculate(array[0],array[1]);  
    }  
      
    /*被子类重写的方法*/  
    abstract public int calculate(int num1,int num2);  
      
    public int[] split(String exp,String opt){  
        String array[] = exp.split(opt);  
        int arrayInt[] = new int[2];  
        arrayInt[0] = Integer.parseInt(array[0]);  
        arrayInt[1] = Integer.parseInt(array[1]);  
        return arrayInt;  
    }  
}  

//子类
public class Plus extends AbstractCalculator {  
  
	/* 子类实现父类的抽象方法*/
    @Override  
    public int calculate(int num1,int num2) {  
        return num1 + num2;  
    }  
}  

//测试类：
public class StrategyTest {  
  
    public static void main(String[] args) {  
        String exp = "8+8";  
        AbstractCalculator cal = new Plus();  
        int result = cal.calculate(exp, "\\+");  
        System.out.println(result);  
    }  
}  
```

## 15、观察者模式（Observer）

包括这个模式在内的接下来的四个模式，都是类和类之间的关系，不涉及到继承，学的时候应该 记得归纳，记得本文最开始的那个图。观察者模式很好理解，类似于邮件订阅和RSS订阅，当我们浏览一些博客或wiki时，经常会看到RSS图标，就这的意思是，当你订阅了该文章，如果后续有更新，会及时通知你。其实，简单来讲就一句话：当一个对象变化时，其它依赖该对象的对象都会收到通知，并且随着变化！

```java
public interface Observer {  
	/*用于更新观察者本身的属性或执行某些行为*/
    public void update();  
}  

//两个实现类：
public class Observer1 implements Observer {  
  
    @Override  
    public void update() {  
        System.out.println("observer1 has received!");  
    }  
}  

public class Observer2 implements Observer {  
  
    @Override  
    public void update() {  
        System.out.println("observer2 has received!");  
    }  
  
}  

//Subject接口及实现类：
public interface Subject {  
      
    /*增加观察者*/  
    public void add(Observer observer);  
      
    /*删除观察者*/  
    public void del(Observer observer);  
      
    /*通知所有的观察者*/  
    public void notifyObservers();  
      
    /*自身的操作*/  
    public void operation();  
}  

//抽象类实现接口的一些通用操作
public abstract class AbstractSubject implements Subject {  
  
    private Vector<Observer> vector = new Vector<Observer>();  
    @Override  
    public void add(Observer observer) {  
        vector.add(observer);  
    }  
  
    @Override  
    public void del(Observer observer) {  
        vector.remove(observer);  
    }  
  
	/*通知所有观察者，调用观察者的更新方法*/
    @Override  
    public void notifyObservers() {  
        Enumeration<Observer> enumo = vector.elements();  
        while(enumo.hasMoreElements()){  
            enumo.nextElement().update();  
        }  
    }  
}  

//实现类继承抽象类实现接口的具体区别的操作
public class MySubject extends AbstractSubject {  
  
    @Override  
    public void operation() {  
        System.out.println("update self!");  
        notifyObservers();  
    }  
  
}  

测试类：

[java] view plaincopy
public class ObserverTest {  
  
    public static void main(String[] args) {  
        Subject sub = new MySubject();  
        sub.add(new Observer1());  
        sub.add(new Observer2());  
          
        sub.operation();  
    }  
  
}  
```

## 16、迭代子模式（Iterator）

顾名思义，迭代器模式就是顺序访问聚集中的对象，一般来说，集合中非常常见，如果对集合类比较熟悉的话，理解本模式会十分轻松。这句话包含两层意思：一是需要遍历的对象，即聚集对象，二是迭代器对象，用于对聚集对象进行遍历访问。
```java
public interface Collection {  
      
    public Iterator iterator();  
      
    /*取得集合元素*/  
    public Object get(int i);  
      
    /*取得集合大小*/  
    public int size();  
}  

public interface Iterator {  
    //前移  
    public Object previous();  
      
    //后移  
    public Object next();  
    public boolean hasNext();  
      
    //取得第一个元素  
    public Object first();  
}  

//两个实现：
public class MyCollection implements Collection {  
  
    public String string[] = {"A","B","C","D","E"};  
    @Override  
    public Iterator iterator() {  
        return new MyIterator(this);  
    }  
  
    @Override  
    public Object get(int i) {  
        return string[i];  
    }  
  
    @Override  
    public int size() {  
        return string.length;  
    }  
}  

public class MyIterator implements Iterator {  
  
    private Collection collection;  
    private int pos = -1;  
      
    public MyIterator(Collection collection){  
        this.collection = collection;  
    }  
      
    @Override  
    public Object previous() {  
        if(pos > 0){  
            pos--;  
        }  
        return collection.get(pos);  
    }  
  
    @Override  
    public Object next() {  
        if(pos<collection.size()-1){  
            pos++;  
        }  
        return collection.get(pos);  
    }  
  
    @Override  
    public boolean hasNext() {  
        if(pos<collection.size()-1){  
            return true;  
        }else{  
            return false;  
        }  
    }  
  
    @Override  
    public Object first() {  
        pos = 0;  
        return collection.get(pos);  
    }  
  
} 
 
//测试类：
public class Test {  
  
    public static void main(String[] args) {  
        Collection collection = new MyCollection();  
        Iterator it = collection.iterator();  
          
        while(it.hasNext()){  
            System.out.println(it.next());  
        }  
    }  
}  
```

## 17、责任链模式（Chain of Responsibility）
接下来我们将要谈谈责任链模式，有多个对象，每个对象持有对下一个对象的引用，这样就会形成一条链，请求在这条链上传递，直到某一对象决定处理该请求。但是发出者并不清楚到底最终那个对象会处理该请求，所以，责任链模式可以实现，在隐瞒客户端的情况下，对系统进行动态的调整。

```java
public interface Handler {  
    public void operator();  
}  

public abstract class AbstractHandler {  
      
    private Handler handler;  
  
    public Handler getHandler() {  
        return handler;  
    }  
  
    public void setHandler(Handler handler) {  
        this.handler = handler;  
    }  
      
}  

public class MyHandler extends AbstractHandler implements Handler {  
  
    private String name;  
  
    public MyHandler(String name) {  
        this.name = name;  
    }  
  
    @Override  
    public void operator() {  
        System.out.println(name+"deal!");  
        if(getHandler()!=null){  
            getHandler().operator();  
        }  
    }  
}  
```

## 18、命令模式（Command）

命令模式很好理解，举个例子，司令员下令让士兵去干件事情，从整个事情的角度来考虑，司令员的作用是，发出口令，口令经过传递，传到了士兵耳朵里，士兵去执行。这个过程好在，三者相互解耦，任何一方都不用去依赖其他人，只需要做好自己的事儿就行，司令员要的是结果，不会去关注到底士兵是怎么实现的。

```java
public interface Command {  
    public void exe();  
}  

public class MyCommand implements Command {  
  
    private Receiver receiver;  
      
    public MyCommand(Receiver receiver) {  
        this.receiver = receiver;  
    }  
  
    @Override  
    public void exe() {  
        receiver.action();  
    }  
}  

public class Receiver {  
    public void action(){  
        System.out.println("command received!");  
    }  
}  

public class Invoker {  
      
    private Command command;  
      
    public Invoker(Command command) {  
        this.command = command;  
    }  
  
    public void action(){  
        command.exe();  
    }  
}  

public class Test {  
  
    public static void main(String[] args) {  
        Receiver receiver = new Receiver();  
        Command cmd = new MyCommand(receiver);  
        Invoker invoker = new Invoker(cmd);  
        invoker.action();  
    }  
}  
```

## 19、备忘录模式（Memento）

主要目的是保存一个对象的某个状态，以便在适当的时候恢复对象，个人觉得叫备份模式更形象些，通俗的讲下：假设有原始类A，A中有各种属性，A可以决定需要备份的属性，备忘录类B是用来存储A的一些内部状态，类C呢，就是一个用来存储备忘录的，且只能存储，不能修改等操作。

Original类是原始类，里面有需要保存的属性value及创建一个备忘录类，用来保存value值。Memento类是备忘录类，Storage类是存储备忘录的类，持有Memento类的实例，
```java
public class Original {  
      
    private String value;  
      
    public String getValue() {  
        return value;  
    }  
  
    public void setValue(String value) {  
        this.value = value;  
    }  
  
    public Original(String value) {  
        this.value = value;  
    }  
  
    /*创建并返回备忘录*/
    public Memento createMemento(){  
        return new Memento(value);  
    }  
      
    /*使用备忘录回复值*/
    public void restoreMemento(Memento memento){  
        this.value = memento.getValue();  
    }  
}  

public class Memento {  
      
    private String value;  
  
    public Memento(String value) {  
        this.value = value;  
    }  
  
    public String getValue() {  
        return value;  
    }  
  
    public void setValue(String value) {  
        this.value = value;  
    }  
}  

public class Storage {  
      
    private Memento memento;  
      
    public Storage(Memento memento) {  
        this.memento = memento;  
    }  
  
    public Memento getMemento() {  
        return memento;  
    }  
  
    public void setMemento(Memento memento) {  
        this.memento = memento;  
    }  
}  

//测试类：
public class Test {  
  
    public static void main(String[] args) {  
          
        // 创建原始类  
        Original origi = new Original("egg");  
  
        // 创建备忘录  
        Storage storage = new Storage(origi.createMemento());  
  
        // 修改原始类的状态  
        System.out.println("初始化状态为：" + origi.getValue());  
        origi.setValue("niu");  
        System.out.println("修改后的状态为：" + origi.getValue());  
  
        // 回复原始类的状态  
        origi.restoreMemento(storage.getMemento());  
        System.out.println("恢复后的状态为：" + origi.getValue());  
    }  
}  
```


## 20、状态模式（State）

核心思想就是：当对象的状态改变时，同时改变其行为，很好理解！就拿QQ来说，有几种状态，在线、隐身、忙碌等，每个状态对应不同的操作，而且你的好友也能看到你的状态，所以，状态模式就两点：1、可以通过改变状态来获得不同的行为。2、你的好友能同时看到你的变化。

```java
public class State {  
      
    private String value;  
      
    public String getValue() {  
        return value;  
    }  
  
    public void setValue(String value) {  
        this.value = value;  
    }  
  
    public void method1(){  
        System.out.println("execute the first opt!");  
    }  
      
    public void method2(){  
        System.out.println("execute the second opt!");  
    }  
}  

public class Context {  
  
    private State state;  
  
    public Context(State state) {  
        this.state = state;  
    }  
  
    public State getState() {  
        return state;  
    }  
  
    public void setState(State state) {  
        this.state = state;  
    }  
  
    public void method() {  
        if (state.getValue().equals("state1")) {  
            state.method1();  
        } else if (state.getValue().equals("state2")) {  
            state.method2();  
        }  
    }  
}  

//测试类：
public class Test {  
  
    public static void main(String[] args) {  
          
        State state = new State();  
        Context context = new Context(state);  
          
        //设置第一种状态  
        state.setValue("state1");  
        context.method();  
          
        //设置第二种状态  
        state.setValue("state2");  
        context.method();  
    }  
}  
```
> 根据这个特性，状态模式在日常开发中用的挺多的，尤其是做网站的时候，我们有时希望根据对象的某一属性，区别开他们的一些功能，比如说简单的权限控制等。

## 21、访问者模式（Visitor）

访问者模式把数据结构和作用于结构上的操作解耦合，使得操作集合可相对自由地演化。访问者模式适用于数据结构相对稳定算法又易变化的系统。因为访问者模式使得算法操作增加变得容易。若系统数据结构对象易于变化，经常有新的数据对象增加进来，则不适合使用访问者模式。访问者模式的优点是增加操作很容易，因为增加操作意味着增加新的访问者。访问者模式将有关行为集中到一个访问者对象中，其改变不影响系统数据结构。其缺点就是增加新的数据结构很困难。

简单来说，访问者模式就是一种**分离对象数据结构与行为的方法**，通过这种分离，可达到为一个被访问者动态添加新的操作而无需做其它的修改的效果。
- 即新增访问者操作去访问被访问者的属性。

```java
public interface Visitor {  
    public void visit(Subject sub);  
}  

public class MyVisitor implements Visitor {  
  
    @Override  
    public void visit(Subject sub) {  
        System.out.println("visit the subject："+sub.getSubject());  
    }  
} 
 
//Subject类，accept方法，接受将要访问它的对象，getSubject()获取将要被访问的属性，
public interface Subject {  
    public void accept(Visitor visitor);  
    public String getSubject();  
}  

public class MySubject implements Subject {  
  
    @Override  
    public void accept(Visitor visitor) {  
        visitor.visit(this);  
    }  
  
    @Override  
    public String getSubject() {  
        return "love";  
    }  
}  

//测试：
public class Test {  
  
    public static void main(String[] args) {  
          
        Visitor visitor = new MyVisitor();  
        Subject sub = new MySubject();  
        sub.accept(visitor);      
    }  
}  
```

> 该模式适用场景：如果我们想为一个现有的类增加新功能，不得不考虑几个事情：1、新功能会不会与现有功能出现兼容性问题？2、以后会不会再需要添加？3、如果类不允许修改代码怎么办？面对这些问题，最好的解决方法就是使用访问者模式，**访问者模式适用于数据结构相对稳定的系统，把数据结构和算法解耦。**

## 22、中介者模式（Mediator）

中介者模式也是用来降低类类之间的耦合的，因为如果类类之间有依赖关系的话，不利于功能的拓展和维护，因为只要修改一个对象，其它关联的对象都得进行修改。如果使用中介者模式，只需关心和Mediator类的关系，具体类类之间的关系及调度交给Mediator就行，这有点像spring容器的作用
```java
public interface Mediator {  
    public void createMediator();  
    public void workAll();  
}  

public class MyMediator implements Mediator {  
  
    private User user1;  
    private User user2;  
      
    public User getUser1() {  
        return user1;  
    }  
  
    public User getUser2() {  
        return user2;  
    }  
  
    @Override  
    public void createMediator() {  
        user1 = new User1(this);  
        user2 = new User2(this);  
    }  
  
    @Override  
    public void workAll() {  
        user1.work();  
        user2.work();  
    }  
}  

public abstract class User {  
      
    private Mediator mediator;  
      
    public Mediator getMediator(){  
        return mediator;  
    }  
      
    public User(Mediator mediator) {  
        this.mediator = mediator;  
    }  
  
    public abstract void work();  
}

public class User1 extends User {  
  
    public User1(Mediator mediator){  
        super(mediator);  
    }  
      
    @Override  
    public void work() {  
        System.out.println("user1 exe!");  
    }  
}  

public class User2 extends User {  
  
    public User2(Mediator mediator){  
        super(mediator);  
    }  
      
    @Override  
    public void work() {  
        System.out.println("user2 exe!");  
    }  
}  

//测试类：
public class Test {  
  
    public static void main(String[] args) {  
        Mediator mediator = new MyMediator();  
        mediator.createMediator();  
        mediator.workAll();  
    }  
}  
```

## 23、解释器模式（Interpreter）
解释器模式是我们暂时的最后一讲，一般主要应用在OOP开发中的编译器的开发中，所以适用面比较窄。



Context类是一个上下文环境类，Plus和Minus分别是用来计算的实现，代码如下：

 
```java

public interface Expression {  
    public int interpret(Context context);  
}  

public class Plus implements Expression {  
  
    @Override  
    public int interpret(Context context) {  
        return context.getNum1()+context.getNum2();  
    }  
}  

public class Minus implements Expression {  
  
    @Override  
    public int interpret(Context context) {  
        return context.getNum1()-context.getNum2();  
    }  
}  

public class Context {  
      
    private int num1;  
    private int num2;  
      
    public Context(int num1, int num2) {  
        this.num1 = num1;  
        this.num2 = num2;  
    }  
      
    public int getNum1() {  
        return num1;  
    }  
    public void setNum1(int num1) {  
        this.num1 = num1;  
    }  
    public int getNum2() {  
        return num2;  
    }  
    public void setNum2(int num2) {  
        this.num2 = num2;  
    }  
      
      
}  

public class Test {  
  
    public static void main(String[] args) {  
  
        // 计算9+2-8的值  
        int result = new Minus().interpret((new Context(new Plus()  
                .interpret(new Context(9, 2)), 8)));  
        System.out.println(result);  
    }  
}  
```