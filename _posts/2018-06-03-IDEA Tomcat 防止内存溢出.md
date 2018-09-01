---
title: IDEA Tomcat 防止内存溢出
date: 2018-06-03
author: asing1elife
categories:
 - software
 - idea
tags:
 - software
 - idea
---
> IDEA 中运行运行默认配置的 Tomcat 经常会抛出 `OutOfMemoryError： PermGen space` 的错误  

## 错误原因
1. 出现上述错误的原因是因为 Tomcat 默认配置的内存空间非常小
2. 所以在运行较大项目时就会出现内存溢出的错误

## 解决方法
1. 在 **VM options** 中按需增加 `-server -XX:PermSize=256M -XX:MaxPermSize=512m` 的参数即可
![](http://asing1elife.com/sources/images/32031F1E-F0B9-4DB2-BBCC-0E23458EA76F.png)