---
title: MyBatis新增返回id
date: 2017-02-08
author: asing1elife
categories:
 - java
 - mybatis
tags:
 - java
 - mybatis
---
> 新增一条数据到数据库，并返回该数据的id  

## 实现方式
```xml
<insert id="saveSendMessage" keyColumn="id" keyProperty="id" useGeneratedKeys="true" parameterType="SendMessageDTO">
	...
</insert>
```