---
title: SpringBoot多环境配置
date: 2018-05-09
author: asing1elife
categories:
 - java
 - springboot
tags:
  - java
  - springboot
---
> SpringBoot 在开发时可以配置多个环境进行便捷切换  

## 创建多个环境配置文件
![](http://asing1elife.com/sources/images/2D5C30D2-EA97-41C2-B323-4F7A77EFD1F7.png)

1. 首先需要创建多个对应的配置文件，如上图
2. 然后在 **application.yml** 中通过如下语法进行匹配
	* 项目启动时会根据指定的尾缀自动去匹配对应的配置文件

```yaml
spring:
  profiles:
    active: dev
```

## 项目打包实现动态指定配置文件
1. 执行 `java -jar xxx.jar`  会直接按照默认配置进行打包
2. 执行 `java -jar xxx.jar --spring.profiles.active=test` 则可以动态指定配置文件