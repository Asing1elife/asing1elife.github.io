---
title: MyBatis抛出You can't operate on a closed Connection!!!
date: 2018-03-08
author: asing1elife
categories:
 - java
 - mybatis
 - error
tags:
 - java
 - mybatis
 - error
---
> 有时候通过 Session 获取数据库连接时为空  

## 碰到的问题
1. 一般通过以下方式获取数据库连接
2. 但有时候会出现获取不到连接，从而抛出 You can't operate on a closed Connection!!! 的异常

```java
Connection connection = this.SqlSession().getConnection();
```

## 解决的方式
1. 使用以下方式获取数据库连接可保证获取的连接存在与事务中不会莫名丢失

```java
SqlSessionTemplate st = (SqlSessionTemplate) this.getSqlSession();
SqlSession session = SqlSessionUtils.getSqlSession(st.getSqlSessionFactory(), st.getExecutorType(), st.getPersistenceExceptionTranslator());
Connection connection = session.getConnection();
```