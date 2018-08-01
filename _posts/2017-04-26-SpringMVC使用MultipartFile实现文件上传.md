---
title: SpringMVC使用MultipartFile实现文件上传
date: 2017-04-26
author: asing1elife
categories:
 - java
 - springmvc
tags:
 - java
 - springmvc
 - fileupload
---
> SpringMVC 接收 MultipartFile 需要在 spring-mvc.xml 中配置文件解析器  

## 配置
```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="20971520"/>
    <property name="defaultEncoding" value="UTF-8"/>
</bean>
```

## 依赖
```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
</dependency>
<dependency>
	<groupId>commons-io</groupId>
	<artifactId>commons-io</artifactId>
</dependency>
```