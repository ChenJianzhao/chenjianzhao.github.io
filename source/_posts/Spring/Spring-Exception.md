---
categories: Spring
date: 2017-01-12 00:10
status: public
title: 'Spring 常见异常'
---

> Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sessionFactory' defined in class path resource [applicationContext.xml]: Invocation of init method failed; nested exception is org.hibernate.AnnotationException: No identifier specified for entity: org.demo.model.UserInfo

- 没有为 ``UserInfo`` 实体设置 ``@Id `` 注解

---
> @MappedSuperclass 的作用

- pojo作为公有父类被用来继承，不单独作为实体使用时，使用注解``@MappedSuperclass``，这样父类将不独立拥有一张表，子类的表中将包含子类自身以及父类的所有字段


***
> 
web.xml文件报错
servlet 映射报错
Servlet should have a mapping Checks if all servlets have mappings
Cannot resolve Servlet 'springmvc'

- 可能是 web module 没有设置好，web文件路径没有设置正确导致。