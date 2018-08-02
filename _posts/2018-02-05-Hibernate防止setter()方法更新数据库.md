---
title: Hibernate防止setter()方法更新数据库
date: 2018-02-05
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
---
> Hibernate从数据库获取到对象后直接调用其setter()方法对内部数据做更改，可能会导致直接将数据更新至数据库  

## 产生问题的原因
1. Hibernate分为三种基本状态：游离态，自由态，持久态
2. 从数据库中获取到对象属于持久态，直接进行操作会导致处于Session中的数据发生改变，从而触发数据库更新

## 解决办法
1. 获取到当前的Session对象，将该对象从Session中清除

```java
super.getEntityDao().getSession().evict(userWork);
```