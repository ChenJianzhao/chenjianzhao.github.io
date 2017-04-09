---
categories:
  - 'Java Core'
  - 'Java 设计模式'
date: 2017-03-31 00:16
status: public
title: 'Java设计模式（十四）模板方法模式（Template Method）'
---

## 介绍
一个抽象类中，有一个主方法，再定义1...n个方法，可以是抽象的，也可以是实际的方法，主方法调用抽象方法；再定义一个子类，继承该抽象类，重写那1...n个抽象方法，子类通过调用抽象类的主方法，主方法调用抽象方法时会从子类逐级向上寻找具体实现，实现对子类方法的调用。
<!-- more -->
抽象类
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
```
子类
```java
public class Plus extends AbstractCalculator {  
  
	/* 子类实现父类的抽象方法*/
    @Override  
    public int calculate(int num1,int num2) {  
        return num1 + num2;  
    }  
}  
```
测试类
```java
public class StrategyTest {  
  
    public static void main(String[] args) {  
        String exp = "8+8";  
        AbstractCalculator cal = new Plus();  
        int result = cal.calculate(exp, "\\+");  
        System.out.println(result);  
    }  
}  
```