---
categories: Spring
date: 2017-02-17 00:22
status: public
title: 'CXF集成Spring构建Web Service'
---

## CXF和Axis的比较
***
这两个项目都开发不够成熟，但是最主要的区别在以下几个方面： 
1. CXF支持 WS-Addressing，WS-Policy， WS-RM， WS-Security和WS-I Basic Profile。Axis2不支持WS-Policy，但是承诺在下面的版本支持。 
2. CXF可以很好支持Spring。Axis2不能 
3. AXIS2支持更广泛的数据并对，如XMLBeans，JiBX，JaxMe和JaxBRI和它自定义的数据绑定ADB。注意JaxME和JaxBRI都还是试验性的。CXF只支持JAXB和Aegis。在CXF2.1 
4. Axis2支持多语言-除了Java,他还支持C/C++版本。 

注：基于目前系统主要使用Java开发，并且为了集成Spring等原因，我选择了CXF开发Web Service。

## 一、开发基于 CXF 的 Web Service 服务端 
***
**1、创建WebService服务接口**
```java
package org.demo.webServices;

import javax.jws.WebParam;
import javax.jws.WebService;

@WebService
public interface IHelloWorld {
    //@WebParam(name="arg0")可有可无，为了增强可读性
    public String sayHello(@WebParam(name="userName")String text);
}
```

**2、实现以上接口的实现类**
```java
package org.demo.webServices.impl;

import org.demo.webServices.IHelloWorld;
import javax.jws.WebService;

@WebService(endpointInterface="org.demo.webServices.IHelloWorld")
public class HelloWorldImpl implements IHelloWorld {
    public String sayHello(String text) {
        return "Hello" + text ;
    }
}
```

**3、在Web.xml中声明基本的CXF的服务servlet**
同所有的servlet一样，servlet存在的目的就是将web前端的URL请求映射到后台的处理器中去，因此web.xml中需要配置映射原则，这里我们可以看到CXFServlet用于映射所有的URL，其其实可以带有ws标识的URL到自己的处理范围里，也就是说，一旦满足这样模式的URL，都将请求WebService。
```xml
<servlet>
    <servlet-name>CXFServlet</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>CXFServlet</servlet-name>
    <url-pattern>/ws/*</url-pattern>
</servlet-mapping>
```

**4、修改``applicationContext-server.xml``，配置服务信息**
jaxws可以看作是一种特殊的bean，WebService服务实体bean，这是CXF框架为Spring提供的集成支持，在这里我们可以看到，当URL的请求地址为/HelleWorld时，将会调用我们已经注入的服务bean处理这个请求，进而完成外界对系统开放的WebService的调用。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    ...
       xmlns:jaxws="http://cxf.apache.org/jaxws"
       xsi:schemaLocation="
    ...
       http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">

<!--新版的CXF只需要引入只一个配置文件，文件位置 cxf-cor.jar MEAT-INF/cxf/cxf.xml -->    <import resource="classpath:/cxf/cxf.xml" />
<jaxws:endpoint id="helloWorld" implementor="org.demo.webServices.impl.HelloWorldImpl" address="/HelloWorld" />
```

**5、部署服务**
将我们的工程部署到Tomcat服务器中去，启动服务器查看如下URL：http://localhost:8080/ws
在这里浏览器会返回CXF渲染的web页面用以说明当前的``CXFServlet``能够导向的服务有哪些：IHelloWorld sayHello使用``/HelloWorld``的URL就可以调用。
我们再使用这样的URL来查看这个服务具体的定义：http://localhost:8080/ws/HelloWorld?wsdl

![](~/17-49-03.jpg)
![](~/17-52-47.jpg)
至此，服务端开发和发布成功。

## 二、客户端开发
***
**1、根据 WSDL 生产客户端代码**
CXF 发布包自带了根据 WSDL 生产客户端代码的工具
```
wsdl2java http://localhost:8080/ws/HelloWorld?wsdl
```
具体参数的作用可以查看官方文档。

**2、将生产的代码拷贝到客户端系统代码路径下**

![](~/18-17-29.jpg)

**3、配置客户端 applicationContext-client.xml**
在Spring的配置文件中增加Web Service的Bean信息，接受容器管理，用于注入。
**方法一：使用CXF提供的标签``jaxws``**
```xml
<jaxws:client id="helloWorld"
              serviceClass="org.demo.webservices.IHelloWorld"
              address="http://localhost:8080/ws/HelloWorld"/>
```
**方法二：使用CXF的客户端Bean工厂类``JaxWsProxyFactoryBean``创建Bean**
```xml
<bean id="client" class="org.demo.webservices.IHelloWorld" factory-bean="clientFactory"
    factory-method="create" />
    
<bean id="clientFactory" class="org.apache.cxf.jaxws.JaxWsProxyFactoryBean">
    <property name="serviceClass" value="org.demo.webservices.IHelloWorld" />
    <property name="address"
        value="http://localhost:8080/ws/HelloWorld" />
</bean>
```
**4、编写客户端测试代码**
```java
import org.demo.webservices.IHelloWorld;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import javax.annotation.Resource;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class TestWebService {

    @Resource
    public IHelloWorld helloWorld = null;

    @Test
    public void testHelloWorld() {
        System.out.println(helloWorld.sayHello("test"));;
    }
```
客户端调用结果
![](~/18-27-00.jpg)