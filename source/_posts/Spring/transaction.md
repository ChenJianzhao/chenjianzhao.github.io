---
categories: Spring
date: 2017-01-15 23:31
status: public
title: 'Spring 声明式事务'
---

## 一、配置文件总览
***
```xml
 <!-- 第一步：创建Spring的sessionFactory  -->
    <bean id="sessionFactory" class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
        <!-- 注入数据源  -->
        <property name="dataSource" ref="dataSource" />
        <!-- 设置Spring去哪个包中查找相应的实体类   -->
        <property name="packagesToScan" value="org.demo.model" />
        <!-- 代替hibernate-config 文件中的设置   -->
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
                <prop key="hibernate.format_sql">false</prop>
            </props>
        </property>
    </bean>

    <!-- 配置spring的事务处理 -->
    <bean id="txManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
    
    <!--第三步：配置事务规则-->
    <!-- 方法一：全注解配置法，打开注解功能, 然后用@Transactional对类或者方法进行标记，如果标记到类上，那么此类中所有方法都进行事务回滚处理，在类中定义Transactional的时候，它有propagation、rollbackFor、noRollbackFor等属性，此属性是用来定义事务规则，而定义到哪这个就是事务入口。-->
    <!--<tx:annotation-driven transaction-manager="txManager" />-->
    <!-- 方法二，xml方式 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <!-- 配置事务传播特性 -->
        <tx:attributes>
            <!-- 为了提高效率,可以把一些查询之类的方法设置为只读事务  -->
            <!--<tx:method name="*" propagation="REQUIRED"  read-only="true"/>-->
            <tx:method name="*" propagation="REQUIRED" rollback-for="true"/>
        </tx:attributes>
    </tx:advice>
    
    <!-- 第四步：配置aop，Spring是通过aop来进行事务管理的  -->
    <!-- 设置pointcut 表明哪些方法需要加入事务   -->
    <!-- 以下的事务是声明在Dao中的，但是通常都会在Service来处理多个业务对象的逻辑关系
    如删除、更新等，此时如果在执行一个步骤之后抛出异常，就会导致数据不完整，所以事务
    不应该在Dao层 处理，应该在Service，使用spring的声明式事务。
      -->

    <aop:config>
        <aop:pointcut id="allServiceMethod" expression="execution(* org.demo.service.*.*(..))" />
        <aop:advisor pointcut-ref="allServiceMethod" advice-ref="txAdvice" />
    </aop:config>
```


## 二、关于 hibernate.hbm2ddl.auto 参数
***
其实这个 ``hibernate.hbm2ddl.auto`` 参数的作用主要用于：自动创建|更新|验证数据库表结构。如果不是此方面的需求建议set value=”none”。 
``create``：
每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。 
``create-drop ``： 
每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。 
``update``： 
最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据 model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等 应用第一次运行起来后才会。 
``validate ``： 
每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。


## 三、常用 hibernate properties
***
``hibernate.dialect ``;一个Hibernate Dialect类名允许Hibernate针对特定的关系数据库生成优化的SQL. 取值 full.classname.of.Dialect 
``hibernate.show_sql``;输出所有SQL语句到控制台. 有一个另外的选择是把org.hibernate.SQL这个log category设为debug。 eg. true | false 
``hibernate.format_sql`` 在log和console中打印出更漂亮的SQL。 取值 true | false 
``hibernate.default_schema`` 在生成的SQL中, 将给定的schema/tablespace附加于非全限定名的表名上. 取值 SCHEMA_NAME 
``hibernate.default_catalog`` 在生成的SQL中, 将给定的catalog附加于非全限定名的表名上. 取值 CATALOG_NAME 
``hibernate.session_factory_name`` SessionFactory创建后，将自动使用这个名字绑定到JNDI中. 取值 jndi/composite/name 
``hibernate.max_fetch_depth`` 为单向关联(一对一, 多对一)的外连接抓取（outer join fetch）树设置最大深度. 值为0意味着将关闭默认的外连接抓取. 取值 建议在0到3之间取值 
``hibernate.default_batch_fetch_size`` 为Hibernate关联的批量抓取设置默认数量. 取值 建议的取值为4, 8, 和16 
``hibernate.default_entity_mode`` 为由这个SessionFactory打开的所有Session指定默认的实体表现模式. 取值 dynamic-map, dom4j, pojo 
``hibernate.order_updates`` 强制Hibernate按照被更新数据的主键，为SQL更新排序。这么做将减少在高并发系统中事务的死锁。 取值 true | false 
``hibernate.generate_statistics`` 如果开启, Hibernate将收集有助于性能调节的统计数据. 取值 true | false 
``hibernate.use_identifer_rollback`` 如果开启, 在对象被删除时生成的标识属性将被重设为默认值. 取值 true | false 
``hibernate.use_sql_comments`` 如果开启, Hibernate将在SQL中生成有助于调试的注释信息, 默认值为false. 取值 true | false 


## 四、tx:method 属性
***
|  属性     |必选| 默认值|描述|
|:---------:|:--:|:-----:|:--:|
|name       | 是 |与事务属性关联的方法名。|通配符（*）可以用来指定一批关联到相同的事务属性的方法。如：'get*'、'handle*'、'on*Event'等等。
|propagation| 不 |REQUIRED   |事务传播行为 |
|isolation	| 不 |DEFAULT    |事务隔离级别 |
|timeout    | 不 |-1	     |事务超时的时间（以秒为单位）|
|read-only	| 不 |	false	 |事务是否只读？|
|rollback-for|不 |		     |将被触发进行回滚的 Exception(s)；以逗号分开。 如：'com.foo.MyBusinessException,ServletException'|
|no-rollback-for|不|         |被触发进行回滚的 Exception(s)；以逗号分开。 如：'com.foo.MyBusinessException,ServletException'

## 五、关于 propagation
***
1. ``PROPAGATION_REQUIRED``: 如果存在一个事务，则支持当前事务。如果没有事务则开启
2. ``PROPAGATION_SUPPORTS``: 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行
3. ``PROPAGATION_MANDATORY``: 如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。
4. ``PROPAGATION_REQUIRES_NEW``: 总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。
5. ``PROPAGATION_NOT_SUPPORTED``: 总是非事务地执行，并挂起任何存在的事务。
6. ``PROPAGATION_NEVER``: 总是非事务地执行，如果存在一个活动事务，则抛出异常 -->