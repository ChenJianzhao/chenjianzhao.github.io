---
categories:
  - 'Java Core'
  - 'Java 进阶'
date: 2016-12-20 22:13
status: public
title: 泛型接口和泛型方法
---

## 泛型接口
> 在泛型接口中，生成器是一个很好的理解，看如下的生成器接口定义：
```java
public interface Generator<T> {
    public T next();
}
```
然后定义一个生成器类来实现这个接口：
```java
public class FruitGenerator implements Generator<String> {
    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```
调用：
```java
public class Main {

    public static void main(String[] args) {
        FruitGenerator generator = new FruitGenerator();
        System.out.println(generator.next());
        System.out.println(generator.next());
        System.out.println(generator.next());
        System.out.println(generator.next());
    }
}
```
输出：
```
Banana
Banana
Pear
Banana
```

</br>

## 泛型方法

泛型方法
> 一个基本的原则是：无论何时，只要你能做到，你就应该尽量使用泛型方法。也就是说，如果使用泛型方法可以取代将整个类泛化，那么应该有限采用泛型方法。下面来看一个简单的泛型方法的定义：
```java
public class Main {

    public static <T> void out(T t) {
        System.out.println(t);
    }

    public static void main(String[] args) {
        out("findingsea");
        out(123);
        out(11.11);
        out(true);
    }
}
```
> 可以看到方法的参数彻底泛化了，这个过程涉及到编译器的类型推导和自动打包，也就说原来需要我们自己对类型进行的判断和处理，现在编译器帮我们做了。这样在定义方法的时候不必考虑以后到底需要处理哪些类型的参数，大大增加了编程的灵活性。

再看一个泛型方法和可变参数的例子：
```java
public class Main {

    public static <T> void out(T... args) {
        for (T t : args) {
            System.out.println(t);
        }
    }

    public static void main(String[] args) {
        out("findingsea", 123, 11.11, true);
    }
}
```