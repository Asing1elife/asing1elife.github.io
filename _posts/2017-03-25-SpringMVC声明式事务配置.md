---
title: SpringMVC声明式事务配置
date: 2017-03-25
author: asing1elife
categories:
 - java
 - springmvc
tags:
 - java
 - springmvc
 - transaction
---
> Spring 的事务管理机制用处在于保证数据统一性  
> 在事务执行过程中如果出现报错，Spring 的事务管理机制会回滚事务，保证数据库的数据不会出现只更改一部分的情况  
> **但触发事务回滚的报错必须是 RuntimeExpection**，只有这种异常可以被 Spring 事务捕获到  
> 如果随意使用 try-catch 对报错进行包裹，会导致报错在方法体内被处理  
> 报错信息不再向上抛出，即 Spring 捕获不到关键的 RuntimeExpection  

## 根据持久层类型选择正确的事务管理器进行配置
* MyBatis 采用的是 JDBC 的事务管理器
* Hibernate 采用的是 Hibernate 自己的事务管理器
* dataSource 是数据源

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource"/>
</bean>
```

## 配置声明式事务
* 默认使用注解管理事务行为，注解也是管理实务的最佳行为

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

## 通过 @Transactional 指定需要事务管理的方法体
* 事务注解最好是打到具体的方法体上，如果打到类名上则意味着整个类都需要进行事务控制，多余的事务控制会影响事务的执行时间
* 方法体中存在增删改操作才需要事务控制
* 只读操作不需要事务控制

``` java
@Transactional
public void executeSkill() { 
... 
}
```