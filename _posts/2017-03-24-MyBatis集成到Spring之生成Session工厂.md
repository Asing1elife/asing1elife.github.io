---
title: MyBatis集成到Spring之生成Session工厂
date: 2017-03-24
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
> 虽然 Spring 自身没有对 MyBatis 提供支持  
> 但 MyBatis 主动对 Spring 进行了整合依赖  

## 依赖
```xml
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis-spring</artifactId>
<version>1.2.2</version>
</dependency>
```

## 配置
1. 在 Spring 中集成 MyBatis 需要使用 SqlSessionFactoryBean 来生成 MyBatis 所需的 Session 工厂

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
...
</bean>
```
2. SqlSessionFactoryBean 实际上并不是真正的 Session 工厂，其经历了如下转换

```java
SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
SqlSessionFactory sessionFactory = factoryBean.getObject();
```

## 属性
1. 指定数据源
	* 该数据源通常引用的是一个数据库连接池的配置项别名
	* `<property name="dataSource" ref="dataSource" />`
2. MyBatis 全局配置文件
	* mybatis-config.xml 中的内容是 [MyBatis settings 配置表](http://asing1elife.com/java/mybatis/2017/03/22/MyBatis的settings配置表/)
	* `<property name="configLocation" value="classpath:mybatis-config.xml" />`
3. 自动扫描 Mapper 文件
	* `<property name="mapperLocations" value="classpath:mapper/*Mapper.xml" />`