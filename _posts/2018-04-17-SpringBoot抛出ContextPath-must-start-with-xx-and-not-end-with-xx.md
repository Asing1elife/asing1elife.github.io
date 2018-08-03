---
title: SpringBoot抛出ContextPath must start with xx and not end with xx异常
date: 2018-04-17
author: asing1elife
categories:
 - java
 - springboot
 - error
tags:
 - java
 - springboot
 - error
---
> 该异常属于项目配置的根路径出错  

## 解决问题的办法
1. 在 `application.yml` 中将 **server.servlet.context-path** 设置的路径前加一个 **/**

```yaml
server:
  servlet:
    context-path: /api
```
