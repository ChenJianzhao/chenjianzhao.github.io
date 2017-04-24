---
categories: MyBatis
date: 2017-04-25 00:22
title: 'MyBatis-Generator'
---
> 原文 [MyBatis Generator 详解](http://blog.csdn.net/isea533/article/details/42102297) 先膜拜一下大神

> [MyBatis Generator中文文档](http://mbg.cndocs.tk/) 该大神翻译的，个人觉得有些地方翻译的不是特别好，觉得拗口的地方建议直接看原文档



配置文件需要命名为 `generatorConfig.xml` 才能被 MyBatis-Generator 识别到

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>

    <!--
    generatorConfiguration 包含以下子元素（有严格的顺序）：
        1. <properties> (0个或1个)
        2. <classPathEntry> (0个或多个)
        3. <context> (1个或多个)
    -->

    <!-- 【1】<properties> (0个或1个)-->
    <properties resource="./jdbc.properties" />

    <!--
        【2】<classPathEntry> (0个或多个)
        最常见的用法是通过这个属性指定驱动的路径
    -->
    <classPathEntry location="C:\Users\jzchen\.m2\repository\mysql\mysql-connector-java\6.0.6\mysql-connector-java-6.0.6.jar"/>

    <!--
        【3】<context> (1个或多个)
        <context>元素用于指定生成一组对象的环境。例如指定要连接的数据库，要生成对象的类型和要处理的数据库中的表。
        运行MBG的时候还可以指定要运行的<context>。
    -->
    <!--
        【必选属性】
            id: 用来唯一确定一个<context>元素，该id属性可以在运行MBG的使用。
        【可选属性】
            1. defaultModelType: 这个属性很重要，这个属性定义了MBG如何生成实体类。
                - conditional:（默认值）,和的hierarchical类似，不同：若只包含一个字段，将不会生成一个单独的类。
                - flat:         （推荐）该模型为每一张表只生成一个实体类。
                - hierarchical： 主键会产生一个单独的主键实体类,所有BLOB字段的生成一个单独的实体类,其他的字段生成一个单独的实体类。 MBG会在所有生成的实体类之间维护一个继承关系。
            2. targetRuntime: MyBatis3(这是默认值) | MyBatis3Simple(不生成和Example) | Ibatis2Java2 | Ibatis2Java5
            3. introspectedColumnImpl:
     -->
    <context id="default" defaultModelType="flat" targetRuntime="MyBatis3" >
        
        <!--
            context 的子元素
            【3.1】<property> (0个或多个)
            下面继续看<property>支持的属性：
                1. autoDelimitKeywords  自动给表名或字段名添加"分隔符"
                2. beginningDelimiter   默认值为双引号(")，可重新指定，MySQL需要 (')
                3. endingDelimiter      默认值为双引号(")，可重新指定
                4. javaFileEncoding     使用当前平台的编码，可重新指定
                5. javaFormatter        使用模板来定制生成的java文件和xml文件的样式
                6. xmlFormatter
        -->
        <!--<property name="autoDelimitKeywords" value="true" />-->
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!--【3.2】<plugin> 元素,0个或者多个 -->
        <!--
            【3.3】<commentGenerator> 元素, 该元素最多可以配置1个。
            可选属性type,可指定用户的实现类，该类需要实现org.mybatis.generator.api.CommentGenerator接口。
            如：<commentGenerator type="com.github.abel533.mybatis.generator.MyCommentGenerator"/>

            默认的特殊值DEFAULT，会使用默认的实现类 org.mybatis.generator.internal.DefaultCommentGenerator。
            可选属性，需要通过<property>属性进行配置。
                1. suppressAllComments: 阻止生成注释，默认为false
                2. suppressDate: 阻止生成的注释包含时间戳，默认为false
        -->
        <commentGenerator>
            <!--<property name="suppressAllComments" value="true"/>-->
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <!--
            【3.4】<jdbcConnection> 必选，并且只能有一个
        -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test?serverTimezone=UTC"
                        userId="root"
                        password="">
        </jdbcConnection>

        <!--
            【3.5】<javaTypeResolver> 元素，最多可以配置一个。
                可以配置的属性为forceBigDecimals，该属性可以控制是否强制
                DECIMAL和 NUMERIC 类型的字段转换为Java类型的 java.math.BigDecimal,
                默认值为false，一般不需要配置。
        -->
        <javaTypeResolver>
            <!--<property name="forceBigDecimals" value="true" />-->
        </javaTypeResolver>

        <!--
            【3.6】<javaModelGenerator> 元素，必须配置一个，并且最多一个。
            该元素用来控制生成的实体类，根据<context>中配置的defaultModelType，前面也推荐使用flat，这种情况下一个表对应一个实体类。
            必选属性
                1. targetPackage: 生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(<table>中会提到)。
                2. targetProject: 指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/java"）。

            <property>子元素属性：
                1. constructorBased:该属性只对MyBatis3有效，如果true就会使用构造方法入参，默认为false。
                2. enableSubPackages:如果true，MBG会根据catalog和schema来生成子包。默认为false。
                3. immutable:该属性用来配置实体类属性是否可变，如果设置为true，那么constructorBased不管设置成什么，都会使用构造方法入参，并且不会生成setter方法。默认为false。
                4. rootClass:设置所有实体类的基类。如果设置，需要使用类的全限定名称。并且如果MBG能够加载rootClass，那么MBG不会覆盖和父类中完全匹配的属性。
                5. trimStrings:是否对数据库查询结果进行trim操作，如果设置为true就会生成类似这样public void setUsername(String username) {this.username = username == null ? null : username.trim();}的setter方法。默认值为false。
        -->
        <javaModelGenerator targetPackage="org.demo.model.mbg" targetProject="src\main\java">
            <property name="constructorBased" value="false" />
            <property name="enableSubPackages" value="false" />
            <property name="trimStrings" value="false" />
        </javaModelGenerator>

        <!--
            【3.7】 <sqlMapGenerator> 元素，该元素可选，最多配置一个
            如果targetRuntime目标是 MyBatis3，只有当<javaClientGenerator>需要XML时，该元素必须配置一个。
            如果没有配置<javaClientGenerator>，则使用以下的规则：
                - 如果指定了一个<sqlMapGenerator>，那么MBG将只生成XML的SQL映射文件和实体类。
                - 如果没有指定<sqlMapGenerator>，那么MBG将只生成实体类。

            必选属性
                1. targetPackage:生成实体类存放的包名，一般就是放在该包下。实际还会受到其他配置的影响(<table>中会提到)。
                2. targetProject:指定目标项目路径，可以是绝对路径或相对路径（如 targetProject="src/main/resources"）。
            <property>子元素，只有一个可以配置的属性：
                1.enableSubPackages:如果true，MBG会根据catalog和schema来生成子包。默认为false。
        -->
        <sqlMapGenerator targetPackage="org.demo.mapper.mbg"  targetProject="src\main\resources">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>

        <!--
            【3.8】<javaClientGenerator> 元素,该元素可选，最多配置一个。如果不配置该元素，就不会生成Mapper接口。
            type:该属性用于选择一个预定义的客户端代码（可以理解为Mapper接口）生成器，用户可以自定义实现，
            需要继承org.mybatis.generator.codegen.AbstractJavaClientGenerator类，必选有一个默认的构造方法。
             该属性提供了以下预定的代码生成器，
            根据<context>的targetRuntime分成三类：
                1. MyBatis3:
                    - ANNOTATEDMAPPER:基于注解的Mapper接口，不会有对应的XML映射文件
                    - MIXEDMAPPER:XML和注解的混合形式，(上面这种情况中的)SqlProvider注解方法会被XML替代。
                    - XMLMAPPER:所有的方法都在XML中，接口调用依赖XML文件。
                2. MyBatis3Simple:
                    - ANNOTATEDMAPPER:基于注解的Mapper接口，不会有对应的XML映射文件
                    - XMLMAPPER:所有的方法都在XML中，接口调用依赖XML文件。
                3. Ibatis2Java2 或 Ibatis2Java5: 略
            该元素支持<property>子元素设置的属性：
                略
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="org.demo.mapper.mbg"
                             targetProject="src\main\java"/>

        <!--
            【3.9】<table> 元素,至少要配置一个，可以配置多个。
            该元素用来配置要通过内省的表。只有配置的才会生成实体类和其他文件。
            该元素有一个必选属性：
                tableName：指定要生成的表名，可以使用SQL通配符匹配多个表。
            <property>子元素，可选属性为：
                1. constructorBased:和<javaModelGenerator>中的属性含义一样。
                2. ignoreQualifiersAtRuntime:生成的SQL中的表名将不会包含schema和catalog前缀。
                3. immutable:和<javaModelGenerator>中的属性含义一样。
                4. modelOnly:此属性用于配置是否为表只生成实体类。如果设置为true就不会有Mapper接口。
                            如果配置了<sqlMapGenerator>，并且modelOnly为true，那么XML映射文件中只有实体对象的映射元素(<resultMap>)。
                            如果为true还会覆盖属性中的enableXXX方法，将不会生成任何CRUD方法。
                其他：略

            <table>还包含以下子元素：
                1. <generatedKey> (0个或1个)
                    这个元素用来指定自动生成主键的属性（identity字段或者sequences序列）。如果指定这个元素，MBG在生成insert的SQL映射文件中插入一个<selectKey>元素。 这个元素**非常重要**，这个元素包含下面两个必选属性：
                        - column:生成列的列名。
                        - sqlStatement:将返回新值的 SQL 语句。如果这是一个identity列，您可以使用其中一个预定义的的特殊值。
                2. <columnRenamingRule> (0个或1个)
                    使用该元素可以在生成列之前，对列进行重命名。这对那些存在同一前缀的字段想在生成属性名时去除前缀的表非常有用。
                3. <columnOverride> (0个或多个)
                4. <ignoreColumn> (0个或多个)
         -->
        <!--
            tableName使用 % 通配符，domainObjectName 无法去除表名的前缀，暂时未找到解决方法
            <table tableName="t\\_%"  schema="test">
                <generatedKey column="id" sqlStatement="Mysql"/>
                <columnRenamingRule searchString="^F" replaceString="" />
            </table>
        -->
        <table tableName="t_blog"  schema="test" domainObjectName="Blog">
            <generatedKey column="id" sqlStatement="Mysql"/>
            <columnRenamingRule searchString="^F" replaceString="" />
            <!--<columnOverride column="FAuthorID" property="author" javaType="User" />-->
        </table>
        <table tableName="t_user"  schema="test" domainObjectName="User">
            <generatedKey column="id" sqlStatement="Mysql"/>
            <columnRenamingRule searchString="^F" replaceString="" />
        </table>
    </context>
</generatorConfiguration>
```



综合以上信息，这里给出一个`Mysql`的简单配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="MysqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <commentGenerator>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/test"
                        userId="root"
                        password="">
        </jdbcConnection>

        <javaModelGenerator targetPackage="test.model" targetProject="src\main\java">
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="test.xml"  targetProject="src\main\resources"/>

        <javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"  targetProject="src\main\java"/>

        <table tableName="%">
            <generatedKey column="id" sqlStatement="Mysql"/>
        </table>
    </context>
</generatorConfiguration>
```