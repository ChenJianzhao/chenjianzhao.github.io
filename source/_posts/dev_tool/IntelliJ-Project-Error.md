---
categories: 开发工具
date: 2016-12-17 20:04
status: public
title: 'IntelliJ 项目异常'
---

> idea @Override is not allowed when implementing interface method

- jdk6 以上的版本才支持 @override 注解实现接口的方法，需要设置两个地方

![](~/20-22-39.jpg) 
![](~/20-05-55.jpg)

![](~/00-58-14.jpg)

> 导入了一个idea project ，编译运行时候，提示Error:java: Compilation failed: internal java compiler error。

- 首先查看 `Project`和 `Modules` 的 jdk 版本，有可能最后是需要设置 `Setting`->`Compiler`->`Java Compiler` Module 的 jdk 版本一直。

***
> 非法字符: '\ufeff' 解决方案|错误: 需要class, interface或enum

-  将报错文件的编码转换为 utf-8 无 BOM