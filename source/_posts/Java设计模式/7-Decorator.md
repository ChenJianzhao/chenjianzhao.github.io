---
categories:
  - 'Java Core'
  - 'Java 设计模式'
date: 2017-03-29 20:51
status: public
title: Java设计模式（七）装饰器模式（Decorator）
---

## 介绍
***
顾名思义，装饰模式就是给一个对象增加一些新的功能，而且是**动态的**，**要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例**。
实现描述：旧类已经实现了接口，装饰器同样实现该接口，创建一个旧类的对象，作为参数传入到装饰器的对象，调用装饰器类的方法，实现功能增强。
（在不改变旧类代码的前提下，装饰原类的每一个方法，实现功能增强）
<!-- more -->
```java
public interface Sourceable {
    public void method();
}
```
```java
public class Source implements Sourceable {

    @Override
    public void method() {
        System.out.println("the original method!");
    }
}
```
```java
public class Decorator implements Sourceable {

    private Sourceable source;

    public Decorator(Sourceable source){
        super();
        this.source = source;
    }
    @Override
    public void method() {
        System.out.println("before decorator!");
        source.method();
        System.out.println("after decorator!");
    }
}
```
测试类：
```java
public class DecoratorTest {

    public static void main(String[] args) {
        Sourceable source = new Source();
        Sourceable obj = new Decorator(source);
        obj.method();
    }
}
```

## 装饰器模式的应用场景
***
1. 需要扩展一个类的功能。
2. 动态的为一个对象增加功能，而且还能动态撤销。（继承不能做到这一点，继承的功能是静态的，不能动态增删。）
   缺点：产生过多相似的对象，不易排错！

## 适配器模式和装饰器模式的区别
***
1. 适配器模式的旧类**没有实现目标接口**，为了重用旧类的代码，将旧类或者旧类的对象包裹在适配器中，调用的依旧是旧类的代码。
2. 装饰器模式的旧类本身**已经实现了目标接口**，只是为了动态拓展类的功能，（如在调用方法的前后输出日志等），用装饰器类包裹旧类，调用旧类的代码。