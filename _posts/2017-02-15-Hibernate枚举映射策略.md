---
title: Hibernate枚举映射策略
date: 2017-02-15
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
 - enum
---
> 稍微解释一下Hibernate枚举策略

## 枚举类型映射到数据库的方式
1. int，即获取枚举的索引值存入数据库，从0开始
2. String，即获取枚举的name属性存入数据库
 
## Hibernate默认把枚举类型的字段当做基本类型(int)的字段来映射
1. 可以通过注解的方式进行控制
    1. `@Enumerated(EnumType.ORDINAL)`  默认方式，int型
    2. `@Enumerated(EnumType.STRING)`  String型，会获取枚举的name属性