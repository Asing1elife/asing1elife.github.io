---
title: 连接 MySQL 抛出 Establishing SSL connection 警告
date: 2018-07-11
author: asing1elife
categories:
 - java
 - database
tags:
 - java
 - mysql
---
> Java 连接 MySQL 时控制台出现多条 Establishing SSL connection without 警告  

## 解决办法
1. 在连接数据库的 url 最后加上 `useSSL=false` 即可
```xml
jdbc:mysql://localhost:3306/hts_lb?characterEncoding=UTF-8&tinyInt1isBit=false&useSSL=false
```