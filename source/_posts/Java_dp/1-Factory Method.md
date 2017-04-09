---
categories:
  - 'Java Core'
  - 'Java 设计模式'
date: 2017-03-29 19:09
status: public
title: 'Java设计模式（一）工厂方法模式（Factory Method）'
---

## 介绍

**工厂方法模式分为三种：**
1. 普通工厂模式

2. 多个工厂方法模式

3. 静态工厂方法模式

<!-- more -->

## 1、普通工厂模式
***
普通工厂模式，就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。
接口
```java
public interface Sender {  
    public void Send();  
}  
```
实现类
```java
public class MailSender implements Sender {  
    @Override  
    public void Send() {  
        System.out.println("this is mailsender!");  
    }  
}  
```
```java
public class SmsSender implements Sender {  
  
    @Override  
    public void Send() {  
        System.out.println("this is sms sender!");  
    }  
}  
```
工厂类：

```java
public class SendFactory {  
  
    public Sender produce(String type) {  
        if ("mail".equals(type)) {  
            return new MailSender();  
        } else if ("sms".equals(type)) {  
            return new SmsSender();  
        } else {  
            System.out.println("请输入正确的类型!");  
            return null;  
        }  
    }  
}  
```
测试类
```java
public class FactoryTest {  
  
    public static void main(String[] args) {  
        SendFactory factory = new SendFactory();  
        Sender sender = factory.produce("sms");  
        sender.Send();  
    }  
}  
```

## 2、多个工厂方法模式
***
多个工厂方法模式，是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象
工厂类
```java
public class SendFactory { 
    public Sender produceMail(){  
        return new MailSender();  
    }  
      
    public Sender produceSms(){  
        return new SmsSender();  
    }  
}  
```
测试类
```java
public class FactoryTest {  
  
    public static void main(String[] args) {  
        SendFactory factory = new SendFactory();  
        Sender sender = factory.produceMail();  
        sender.Send();  
    }  
}  
```

## 3、静态工厂方法模式
***
静态工厂方法模式，将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。
工厂类
```java
public class SendFactory {  
    //静态工厂方法
    public static Sender produceMail(){  
        return new MailSender();  
    }  
    //静态工厂方法
    public static Sender produceSms(){  
        return new SmsSender();  
    }  
}  
```
测试类
```java
public class FactoryTest {  
  
    public static void main(String[] args) {      
        Sender sender = SendFactory.produceMail();  
        sender.Send();  
    }  
}  
```