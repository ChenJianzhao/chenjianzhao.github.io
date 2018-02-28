---
categories: Spring
date: 2018-03-01 07:00
title: Spring 源码浅析（未完）
---



**初始化过程**

DispatcherServlet.init() 【HttpServletBean】

DispatcherServlet.initServletBean()【FrameworkServlet】

FrameworkServlet.initWebApplicationContext()

DispatcherServlet.onRefresh(ApplicationContext)

DispatcherServlet.initStrategies()



**相应请求的过程**

FrameworkServlet.service()

FrameworkServlet.processRequest()

DispatcherServlet.doDispatch()

DispatcherServlet.doDispatch()



**DispatcherServlet 继承体系**

![Spring_sourse_code](Spring_sourse_code/DispatcherServlet.png)

