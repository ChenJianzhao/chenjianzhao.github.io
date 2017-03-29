---
categories: Spring
date: 2017-01-15 19:17
status: public
title: AOP
---

## 导语
***
AOP称为面向切面编程，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等。

## 一、AOP的基本概念
***
(1)``Aspect(切面)``:通常是一个类，里面可以定义切入点和通知
(2)``JointPoint(连接点)``:程序执行过程中明确的点，一般是方法的调用
(3)``Advice(通知)``:AOP在特定的切入点上执行的增强处理，有``before``,``after``,``afterReturning``,``afterThrowing``,``around``
(4)``Pointcut(切入点)``:就是带有通知的连接点，在程序中主要体现为书写切入点表达式
(5)``AOP代理``：AOP框架创建的对象，代理就是目标对象的加强。Spring中的AOP代理可以使JDK动态代理，也可以是CGLIB代理，前者基于接口，后者基于子类

## 二、Spring AOP
***
　　Spring中的AOP代理还是离不开Spring的IOC容器，代理的生成，管理及其依赖关系都是由IOC容器负责，**Spring默认使用JDK动态代理**，在需要代理类而不是代理接口的时候，Spring会自动切换为使用CGLIB代理，不过现在的项目都是面向接口编程，所以JDK动态代理相对来说用的还是多一些。

## 三、基于注解的AOP配置方式
***
### 1. 启用@AsjectJ支持
在applicationContext.xml中配置下面一句:
```xml
<!-- 设置包扫描目录，该包内的所有POJO受 spring 管理   -->
<context:component-scan base-package="org.demo"/>
<!--1.启用 @AsjectJ 支持-->
<aop:aspectj-autoproxy />
```

### 2.通知类型介绍
(1)``Before``:在目标方法被调用之前做增强处理,@Before只需要指定切入点表达式即可
(2)``AfterReturning``:在目标方法**正常完成后做增强**,@AfterReturning除了指定切入点表达式后，还可以指定一个返回值形参名returning,代表目标方法的返回值
(3)``AfterThrowing``:主要用来处理程序中未处理的异常,@AfterThrowing除了指定切入点表达式后，还可以指定一个throwing的返回值形参名,可以通过该形参名
来访问目标方法中所抛出的异常对象
(4)``After``:在目标方法完成之后做增强，**无论目标方法时候成功完成**。@After可以指定一个切入点表达式
(5)``Around``:环绕通知,在目标方法完成前后做增强处理,环绕通知是最重要的通知类型,像事务,日志等都是环绕通知,注意编程中核心是一个 ``ProceedingJoinPoint``

### 3.通知执行的优先级
进入目标方法时,先织入Around,再织入Before，退出目标方法时，先织入Around,再织入After,最后才织入AfterReturning。

### 4.切入点的定义和表达式
切入点表达式的定义算是整个AOP中的核心，有一套自己的规范
**Spring AOP支持的切入点指示符：**
(1)``execution``:用来匹配执行方法的连接点
 ``@Pointcut("execution(* com.aijava.springcode.service..*.*(..))")``
第一个``*``表示匹配任意的方法返回值，``..``(两个点)表示零个或多个，上面的第一个``..``表示service包及其子包,第二个``*``表示所有类,第三个``*``表示所有方法，第二个``..``表示
方法的任意参数个数
(2) ``@Pointcut("within(com.aijava.springcode.service.*)")``
within限定匹配方法的连接点,上面的就是表示匹配service包下的任意连接点
(3) ``@Pointcut("this(com.aijava.springcode.service.UserService)")``
this用来限定AOP代理必须是指定类型的实例，如上，指定了一个特定的实例，就是UserService
(4) ``@Pointcut("bean(userService)")``
bean也是非常常用的,bean可以指定IOC容器中的bean的名称


### 5、基于注解示例代码
**定义切面类**
```java
@Component  //缺少会使切面类不受Spring容器管理，代理将不生效
@Aspect     //注解类为一个切面
public class Operator {

    // 注解切入点表达式
    @Pointcut("execution(* org.demo.service.*.*(..))")
    public void pointCut(){}

    @Before("pointCut()")
    public void doBefore(JoinPoint joinPoint){
        System.out.println("AOP Before Advice...\n");
    }

    @After("pointCut()")
    public void doAfter(JoinPoint joinPoint){
        System.out.println("AOP After Advice...\n");
    }

    @AfterReturning(pointcut="pointCut()",returning="returnVal")
    public void afterReturn(JoinPoint joinPoint,Object returnVal){
        System.out.println("AOP AfterReturning Advice:" + returnVal + "\n");
    }

    @AfterThrowing(pointcut="pointCut()",throwing="error")
    public void afterThrowing(JoinPoint joinPoint,Throwable error){
        System.out.println("AOP AfterThrowing Advice..." + error);
        System.out.println("AfterThrowing...\n");
    }

    @Around("pointCut()")
    public Object around(ProceedingJoinPoint pjp){
        System.out.println("AOP Aronud before...\n");
        Object result = null;
        try {
            result = pjp.proceed(); //此处需要将返回值返回，否则被切入方法将获取不到返回值
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("AOP Aronud after...\n");
        return result;
    }
}
```

**定义业务接口**
```java
public interface IFindUserService {

    public UserInfo findUser(Serializable id);
}
```

**业务接口实现类**
```java
@Service
public class FindUserService implements IFindUserService {

    public UserDao getUserDao() {
        return userDao;
    }

    @Resource
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    private UserDao userDao = null;

    @Override
    public UserInfo findUser(Serializable id) {
        return getUserDao().get(id);
    }
}
```

**测试类**
```java

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:applicationContext.xml")
public class TestAspect {

    @Resource
    IFindUserService findUserService = null;

    public IFindUserService getFindUserService() {
        return findUserService;
    }

    @Resource
    public void setFindUserService(IFindUserService findUserService) {
        this.findUserService = findUserService;
    }

    @Test
    public void testAspect(){
        UserInfo userInfo = getFindUserService().findUser(1);
        System.out.println("out of service : " +  userInfo.getUsername() + " which id is " + userInfo.getId() + "\n");

    }
}
```

**输出结果**
```
AOP Aronud before...
AOP Before Advice...
inner service found user : test which id is 1
AOP Aronud after...
AOP After Advice...
AOP AfterReturning Advice:org.demo.model.UserInfo@4351171a
out of service : test which id is 1
Process finished with exit code 0
```

### 6.基于XML的切面编程
**定义日志类**
```java
public class Log {

    private Integer id;

    //操作名称，方法名
    private String operName;

    //操作人
    private String operator;

    //操作参数
    private String operParams;

    //操作结果 成功/失败
    private String operResult;

    //结果消息
    private String resultMsg;

    //操作时间
    private Date operTime = new Date();

    setter,getter

}
```
**定义Aspect切面类**
```java
/**
 * 日志记录器 （AOP日志通知）
 */
public class Logger {
    
    @Resource
    private LogService logService;
    
    public Object record(ProceedingJoinPoint pjp){
        
        Log log = new Log();
        try {
            log.setOperator("admin");
            String mname = pjp.getSignature().getName();
            log.setOperName(mname);
            
            //方法参数,本例中是User user
            Object[] args = pjp.getArgs();
            log.setOperParams(Arrays.toString(args));
            
            //执行目标方法，返回的是目标方法的返回值，本例中 void
            Object obj = pjp.proceed();
            if(obj != null){
                log.setResultMsg(obj.toString());
            }else{
                log.setResultMsg(null);
            }
            
            log.setOperResult("success");
            log.setOperTime(new Date());
            
            return obj;
        } catch (Throwable e) {
            log.setOperResult("failure");
            log.setResultMsg(e.getMessage());
        } finally{
            logService.saveLog(log);
        }
        return null;
    }
}
```
**applicationContext.xml**
```xml
<aop:config>
    <aop:aspect id="loggerAspect" ref="logger">
    <aop:around method="record" pointcut="execution(* com.aijava.springcode.service..*.*(..)) and !bean(logService)"/>
    </aop:aspect>
</aop:config>
<!-- 注意切入点表达式,!bean(logService) 做日志通知的时候，不要给日志本身做日志，否则会造成无限循环！ -->
```