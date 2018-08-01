---
title: Spring的@Transactional详解
date: 2017-04-05
author: asing1elife
categories:
 - java
 - springboot
 - springmvc
tags:
 - java
 - springboot
 - springmvc
 - spring
 - transaction
---
> Spring 提供的事务管理机制为不同的事务 API 提供一致的编程模型  

## 管理方式
1. 编程式事务
	* TransactionTemplate **[推荐]**
	* PlatformTransactionManager [基于底层]
2. 声明式事务 **[推荐]**
	* 基于 `<tx/>` 和 `<aop/>` 的 xml 配置文件
	* 使用 @Transactional 注解 **[推荐]**

## 配置方式
1. 使用 @Transactional 注解的声明式配置参见 [SpringMVC声明式事务配置](http://asing1elife.com/java/springmvc/2017/03/25/SpringMVC声明式事务配置/)

## 注意点
1. 默认情况下，数据库处于紫铜提交模式，每个语句处于单独的事务中
2. 对于正常的事务管理，应当是一组相关操作处于同一个事务中
3. Spring 默认将底层连接的自动提交设置为 false
4. 有些数据连接池提供自动提交的开关设置，但 c3p0 未提供
5. JDBC 规范明文指出当连接对象建立时应该处理自动提交模式，对于自动提交的开关应该进行显示处理
6. 当一个连接关闭时，未提交的事务应该回滚，虽然 JDBC 规范未明确指出，但 c3p0 默认将 autoCommitOnClose 设置为 false
7. MyBatis 会自动参与到 Spring 的事务管理中，只要二者引用的数据源一致

## @Transactional 属性
1. value - String
	* 限定描述符，用于指定使用的事务管理器
2. isolation - enum
	* 事务隔离级别，参见 [Spring 事务隔离级别](http://asing1elife.com/java/springmvc/springboot/2017/09/14/Spring事务传播行为/)
3. propagation - enum
	* 事务传播行为，参见 [Spring 事务传播行为](http://asing1elife.com/java/springmvc/springboot/2017/09/14/Spring事务传播行为/)
4. readOnly - boolean
	* 事务只读属性，默认读写
	* false = 读写，true = 只读
	* 只读属性用于特殊情景优化，例如在使用 Hibernate 时，默认使用读写事务
5. timeout - int
	* 事务超时时间，单位秒
	* 如果超过该时间事务还没完成，则自动回滚事务
	* 默认设置为底层事务系统的超时值，如果底层未设置，则为 none
6. rollbackFor - Class 对象数组，必须继承自 Throwable
	* 导致事务回滚的异常类数组
7. rollbackForClassName - 类名数组，必须继承自 Throwable
	* 导致事务回滚的异常类名称数组
8. noRollbackFor - Class 对象数组，必须继承自 Throwable
	* 不会导致事务回滚的异常类数组
9. noRollbackForClassName - 类名数组，必须继承自 Throwable
	* 不会导致事务回滚的异常类名称数组
