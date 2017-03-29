---
categories:
  - 'Java Core'
  - 'Java 进阶'
date: 2016-12-25 22:54
status: public
title: Java动态代理
---

## 几个概念
***
**代理**：当程序需要对原始类对象的某个方法进行增强，又不想改写原始类时，可使用代理类实现，避免对旧代码的修改。

**静态代理**：代理类和被代理类实现相同的接口，在代理类的同名方法中，进行增强处理，再调用被代理对象的同名方法。

**静态代理的缺点**：静态代理类和委托类实现了相同的接口，代理类通过委托类实现了相同的方法。这样就出现了大量的代码重复。如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。

在java的动态代理机制中，有两个重要的类或接口，一个是 ``InvocationHandler(Interface)``、另一个则是 ``Proxy(Class)``，这一个类和接口是实现动态代理所必须用到的。

**InvocationHandler 中的 invoke 方法：**
> Object invoke(Object proxy, Method method, Object[] args) throws Throwable
> - roxy: 代理对象本身
> - method: 指代的是我们所要调用真实对象的某个方法的Method对象
> - args: 指代的是调用真实对象某个方法时接受的参数

**Proxy 中的 newProxyInstance 方法**
> public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
> - loader: 一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载
> - interfaces: 一个Interface对象的数组，表示代理对象需要实现的接口，这样就能通过接口声明调用这组接口中的方法了
> - h: 一个InvocationHandler对象，表示这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler 动态代理处理对象上，在该对象中对源方法进行增强。

## 1.首先定义一个被代理类和动态代理类待实现的接口
***
```java
/**
 * Created by jzchen on 2016/12/25 0025.
 */
public interface Source {

    void doSomething();

    String getName();

    String getNumbner();

    String getDrip();
}
```

## 2.被代理的类实现该接口
```java
/**
 * Created by jzchen on 2016/12/25 0025.
 */
public class RealSource implements Source {
    @Override
    public void doSomething() {
        System.out.println("do something here!");
    }

    @Override
    public String getName() {
        System.out.println("getName here!");
        return null;
    }

    @Override
    public String getNumbner() {
        System.out.println("getNumbner here!");
        return null;
    }

    @Override
    public String getDrip() {
        System.out.println("getDrip here!");
        return null;
    }
}
```

## 3.定义动态代理处理类，实现 InvocationHandler 接口，对被代理类方法的调用将转发到此处理类的 invoke 方法中，处理后再调用被代理类真正的方法
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * Created by jzchen on 2016/12/25 0025.
 */
public class ProxyHandler implements InvocationHandler {

    Object realSubject = null;

    ProxyHandler(Object realSubject ){
        this.realSubject = realSubject;
    }


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //do something before invoke real subject 's method
        System.out.println("before method invoke : " + method.getDeclaringClass() + "  :  "+ method.getName() );
        
        Object result = method.invoke(realSubject, args);

        //do something after invoke real subject 's method
        System.out.println("after method invoke !");
        System.out.println("");
        return result;
    }
}
```

## 4.测试类
```java
import java.lang.reflect.Proxy;

/**
 * Created by jzchen on 2016/12/25 0025.
 */
public class TestProxy {

    public static void main(String[] args){

        RealSource realSource = new RealSource();
        Source proxy = (Source)Proxy.newProxyInstance(RealSource.class.getClassLoader(), RealSource.class.getInterfaces(), new ProxyHandler(realSource));

        proxy.doSomething();
        proxy.getNumbner();
        proxy.getName();
        proxy.getDrip();
    }
}
```
## 5.打印结果
```java
before method invoke : interface Source  :  doSomething
do something here!
after method invoke !

before method invoke : interface Source  :  getNumbner
getNumbner here!
after method invoke !

before method invoke : interface Source  :  getName
getName here!
after method invoke !

before method invoke : interface Source  :  getDrip
getDrip here!
after method invoke !

```