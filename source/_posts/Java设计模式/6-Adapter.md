---
categories:
  - 'Java Core'
  - 'Java 设计模式'
date: 2017-03-29 20:36
status: public
title: Java设计模式（六）适配器模式（Adapter）
---

## 介绍
适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是**消除由于接口不匹配所造成的类的兼容性问题**。
**主要分为三类**
1. 类的适配器模式
2. 对象适配器模式
3. 接口的适配器模式
<!-- more -->

## 1. 类的适配器模式
***
实现描述：新类继承旧类同时实现新接口，新类继承了旧类的方法

```java
public class Source {  
  
    public void method1() {  
        System.out.println("this is original method!");  
    }  
}  
```
```java
public interface Targetable {  
  
    /* 与原类中的方法相同 */  
    public void method1();  
  
    /* 新类的方法 */  
    public void method2();  
}  
```
```java
public class Adapter extends Source implements Targetable {  
  
    @Override  
    public void method2() {  
        System.out.println("this is the targetable method!");  
    }  
}  
```
Adapter类继承Source类，实现Targetable接口，下面是测试类：

```java
public class AdapterTest {  
  
    public static void main(String[] args) {  
        Targetable target = new Adapter();  
        target.method1();  
        target.method2();  
    }  
}  
```

## 2. 对象适配器模式
***
实现描述：新类实现借口，new一个旧类的对象，作为参数传入到新类的对象中，新类调用旧类的同名方法时，直接调用旧类对象的该方法。

```java
public class Wrapper implements Targetable {  
  
    private Source source;  
      
    public Wrapper(Source source){  
        super();  
        this.source = source;  
    }  
    @Override  
    public void method2() {  
        System.out.println("this is the targetable method!");  
    }  
  
    @Override  
    public void method1() {  
        source.method1();  
    }  
}  
```
测试类：
```java
public class AdapterTest {  
  
    public static void main(String[] args) {  
        Source source = new Source();  
        Targetable target = new Wrapper(source);  
        target.method1();  
        target.method2();  
    }  
}  
```

## 3. 接口的适配器模式
***
实现描述：创建一个抽象类实现接口的所有方法，新类继承抽象类，只实现需要的方法。

```java
public interface Sourceable {  
      
    public void method1();  
    public void method2();  
}  
```
抽象类Wrapper2：
```java
public abstract class Wrapper2 implements Sourceable{  
      
    public void method1(){}  
    public void method2(){}  
}  
```
```java
public class SourceSub1 extends Wrapper2 {  
    public void method1(){  
        System.out.println("the sourceable interface's first Sub1!");  
    }  
}  
```
```java
public class SourceSub2 extends Wrapper2 {  
    public void method2(){  
        System.out.println("the sourceable interface's second Sub2!");  
    }  
}  
```
```java
public class WrapperTest {  
  
    public static void main(String[] args) {  
        Sourceable source1 = new SourceSub1();  
        Sourceable source2 = new SourceSub2();  
          
        source1.method1();  
        source1.method2();  
        source2.method1();  
        source2.method2();  
    }  
}  
```

## 小结:
***
1. 类的适配器模式：当希望**将一个类转换成满足另一个新接口的类时**，可以使用类的适配器模式，创建一个新类，继承原有的类，实现新的接口即可。
2. 对象的适配器模式：当希望**将一个对象转换成满足另一个新接口的对象时**，可以创建一个Wrapper类，持有原类的一个实例，在Wrapper类的方法中，调用实例的方法就行。
3. 接口的适配器模式：**当不希望实现一个接口中所有的方法时**，可以创建一个抽象类Wrapper，实现所有方法，我们写别的类的时候，继承抽象类即可。