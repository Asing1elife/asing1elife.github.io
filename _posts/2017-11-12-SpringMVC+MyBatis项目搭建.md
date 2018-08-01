---
title: SpringMVC+MyBatis项目搭建
date: 2017-11-12
author: asing1elife
categories:
 - java
 - springmvc
 - mybatis
tags:
 - java
 - springmvc
 - mybatis
---
> SpringMVC作为一个敏捷开发的常用框架，和MyBatis进行集成需要如下步骤  

## 构建项目
1. 创建一个Maven Project，并添加**web.xml**
2. 在**pom.xml**中引入以下依赖包
3. 为保证项目编译的JDK版本统一，需要加入以下配置

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.1</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

## 添加Spring依赖包
```xml
<!-- Spring核心包 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-core</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring IoC -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-beans</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring包扫描 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring数据库连接 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring事务管理器 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-tx</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring Web -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
<!-- Spring WebMVC -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>4.1.6.RELEASE</version>
</dependency>
```

## 添加MyBatis依赖包
```xml
<!-- MyBatis核心包 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.2.7</version>
</dependency>
<!-- MyBatis Spring依赖包 -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>1.2.2</version>
</dependency>
```

## 添加数据库依赖包
```xml
<!-- MySQL连接 -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.18</version>
</dependency>
<!-- c3p0 -->
<dependency>
  <groupId>com.mchange</groupId>
  <artifactId>c3p0</artifactId>
  <version>0.9.5.1</version>
</dependency>
```

## 添加视图层依赖包
```xml
<!-- 视图层的核心包 -->
<dependency>
  <groupId>taglibs</groupId>
  <artifactId>standard</artifactId>
  <version>1.1.2</version>
</dependency>
<!-- JSTL -->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
<!-- Servlet容器 -->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.0.1</version>
</dependency>
```

## 在**web.xml**中添加配置
1. 配置 **DispatcherServlet**

```xml
<!-- 配置DispatcherServlet -->
<servlet>
  <servlet-name>asl-ums-dispatcher</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 加载Spring配置文件 -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-*.xml</param-value>
  </init-param>
  <!-- 项目启动时加载文件 -->
  <load-on-startup>1</load-on-startup>
</servlet>
```

2. 映射路径规则
```xml
<!-- 配置路径映射规则 -->
<servlet-mapping>
  <servlet-name>asl-ums-dispatcher</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

## 创建**spring-mvc.xml**并添加配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="
                http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/mvc
                http://www.springframework.org/schema/mvc/spring-mvc.xsd
http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd">

<!-- 自动注入 -->
<!-- 用于提供数据绑定、数字@NumberFormat和日期格式化@DateTimeFormat以及xml和json的默认读写支持 -->
<mvc:annotation-driven/>

<!-- 配置静态资源 -->
<!-- 加入对js/gif/png等静态资源的处理 -->
<mvc:default-servlet-handler/>

<!-- 指定默认路径 -->
<mvc:view-controller path="/" view-name="redirect:/index"/>

<!-- 配置JSP显示 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/WEB-INF/views"/>
  <property name="suffix" value=".jsp"/>
</bean>

<!-- 扫描指定路径下的类 -->
<context:component-scan base-package="online.shixun.asl.module">
  <!-- 扫描带有@Controller注解的类 -->
  <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
  <!-- 扫描带有@Service注解的类 -->
  <context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
</context:component-scan>

</beans>
```

## 创建**spring-config.xml**并添加配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

<!-- 指定数据库文件 -->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!-- 配置c3p0连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
  <!-- 数据库连接信息 -->
  <property name="driverClass" value="${driver}"/>
  <property name="jdbcUrl" value="${url}"/>
  <property name="user" value="${username}"/>
  <property name="password" value="${password}"/>
  <!-- 最大连接数 -->
  <property name="maxPoolSize" value="30"/>
  <!-- 最小连接数 -->
  <property name="minPoolSize" value="10"/>
  <!-- 获取连接的超时时间，毫秒 -->
  <property name="checkoutTimeout" value="5000"/>
  <!-- 连接失败后的重连次数 -->
  <property name="acquireRetryAttempts" value="1"/>
</bean>

<!-- 配置事务管理器，MyBatis采用jdbc的事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <!-- 指定数据源 -->
  <property name="dataSource" ref="dataSource"/>
</bean>

<!-- 配置声明式事务 -->
<tx:annotation-driven/>

<!-- 配置SqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 指定数据源 -->
  <property name="dataSource" ref="dataSource"/>
  <!-- 指定MyBatis全局配置文件 -->
  <property name="configLocation" value="classpath:spring-mybatis.xml"/>
  <!-- 指定实体类扫描路径 -->
  <property name="typeAliasesPackage" value="online.shixun.asl.dto"/>
  <!-- 指定映射文件的扫描路径 -->
  <property name="mapperLocations" value="classpath:mapper/*Mapper.xml"/>
</bean>

<!-- 将Dao注入Spring容器中 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!-- 指定扫描路径 -->
  <property name="basePackage" value="online.shixun.asl.module"/>
  <!-- 扫描带有@MyBatisRepository -->
  <property name="annotationClass" value="online.shixun.asl.core.MyBatisRepository"/>
</bean>

</beans>
```

## 创建**spring-mybatis.xml**并添加配置
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <!-- 配置全局属性 -->
    <settings>
        <!-- 使用数据库自增主键 -->
        <setting name="useGeneratedKeys" value="true"/>
        <!-- 显示SQL语句 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
</configuration>
```
## 项目配置完毕后，想要启动项目，则需要在Mapper映射路径下至少存在一个mapper.xml