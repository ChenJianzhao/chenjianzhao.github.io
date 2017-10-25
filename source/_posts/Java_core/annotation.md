---
categories:
  - 'Java Core'
  - 'Java 进阶'
date: 2017-03-29 16:22
status: public
title: 'Java Annotation 自定义注解'
---

## 元注解
　　**元注解的作用就是负责注解其他注解。** Java5.0定义了4个标准的`meta-annotation`类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：

1. @Target
2. @Retention
3. @Documented
4. @Inherited

这些类型和它们所支持的类在 **java.lang.annotation** 包中可以找到。下面我们看一下每个元注解的作用和相应分参数的使用说明。

<!-- more -->

## @Target

**``@Target`` 说明了Annotation所修饰的对象范围。**

Annotation可被用于 

- packages（包）
- types（类、接口、枚举、Annotation类型）
- 类型成员（方法、构造方法、成员变量、枚举值）
- 方法参数和本地变量（如循环变量、catch参数）。

**在Annotation类型的声明中使用了target可更加明晰其修饰的目标。**



**作用**：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
**取值：(ElementType)**

1. CONSTRUCTOR:用于描述构造器
2. FIELD:用于描述域
3. LOCAL_VARIABLE:用于描述局部变量
4. METHOD:用于描述方法
5. PACKAGE:用于描述包
6. PARAMETER:用于描述参数
7. TYPE:用于描述类、接口(包括注解类型) 或enum声明




## @Retention

**``@Retention``定义了该Annotation被保留的时间长短。（生命周期）**

某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。



**作用**：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）
**取值（RetentionPoicy）：**

1. SOURCE:在源文件中有效（即源文件保留）
2. CLASS:在class文件中有效（即class保留）
3. RUNTIME:在运行时有效（即运行时保留）



Retention meta-annotation 类型有唯一的value作为成员，它的取值来自 **java.lang.annotation.RetentionPolicy** 的枚举类型值。



## @Documented

**``@Documented``用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，**因此可以被例如javadoc 此类的工具文档化。Documented是一个标记注解，没有成员。




## 示例程序
定义一个注解

```java
package demo.annotation;

import java.lang.annotation.*;

/**
 * 定义一个注解
 */
@Target(ElementType.METHOD) // 这是一个对方法的注解，还可以是包、类、变量等很多东西
@Retention(RetentionPolicy.RUNTIME) // 保留时间，一般注解就是为了框架开发时代替配置文件使用，JVM运行时用反射取参数处理，所以一般都为RUNTIME类型
@Documented // 用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化
public @interface TestAnnotation {

    // 定义注解的参数，类型可以为基本类型以及String、Class、enum、数组等，default为默认值
    String parameter1() default "";
    int parameter2() default -1;
}
```


使用该注解注解类方法

```java
package demo.annotation;

public class Subject {

    @TestAnnotation(parameter1="YES", parameter2=10000)
    public void oneMethod () {
    }
}
```


测试代码

```java
package demo.annotation;

import java.lang.reflect.Method;

public class TestClient {

    public static void main(String[] args) throws Exception {
        // 提取到被注解的方法Method，这里用到了反射的知识
        Method method = Class.forName("demo.annotation.Subject").getDeclaredMethod("oneMethod");
        // 从Method方法中通过方法getAnnotation获得我们设置的注解
        TestAnnotation testAnnotation = method.getAnnotation(TestAnnotation.class);

        // 得到注解的俩参数
        System.out.println(testAnnotation.parameter1());
        System.out.println(testAnnotation.parameter2());
    }
}
```


打印结果

```
YES
10000

Process finished with exit code 0
```