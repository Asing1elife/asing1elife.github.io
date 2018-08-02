---
title: Hibernate 执行 SQL 查询
date: 2018-05-12
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
---
> Hibernate 支持执行原生 SQL 查询  

## 实现方式
1. 通过 `Session` 对象可以使用 `createSQLQuery()` 方法，传入原生 SQL 语句即可
2. 参数格式为 `paramName = :paramCode`

```java
public List<Dictionary> getDictionaries(String categoryCode) {
    SQLQuery sqlQuery = getSession().createSQLQuery("SELECT * FROM sys_dictionary WHERE CategoryCode = :categoryCode");
    sqlQuery.addEntity(Dictionary.class);
    sqlQuery.setParameter("categoryCode", categoryCode);

    return sqlQuery.list();
}
```