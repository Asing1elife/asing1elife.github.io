---
title: Hibernate Criteria 模糊查询
date: 2018-05-14
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
 - criteria
---
> 使用 Hibernate 的 Criteria 可以快速进行模糊查询  

## 实现方式
1. 进行模糊查询时需要指定匹配模式，否则会出现无搜索结果的情况，例如 `MatchMode.ANYWHERE`
2. 如果匹配的条件设置到类的属性，需要使用 `createAlias()` 指定别名，否则会抛出无法找到 `user.name` 属性

```java
Criterion userCri = Restrictions.like("user.name", username, MatchMode.ANYWHERE);

Criteria criteria = getSession().createCriteria(trainingCri, userCri);

criteria.createAlias("user", "user", JoinType.LEFT_OUTER_JOIN);

criteria.list();
```