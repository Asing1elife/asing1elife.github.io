---
title: MyBatis SQL 语句无法输出
date: 2019-03-02
author: asing1elife
categories:
- java
- mybatis
tags:
- java
- mybatis
---
> MyBatis 通过配置文件指定日志输出方式后仍然无法在控制台输出日志  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 异常表现
1. 在配置文件中显式指定了日志输出格式，但控制台依旧不输出SQL语句

```xml
<configuration>
  <settings>
    <setting name="logImpl" value="SLF4J"/>
  </settings>
</configuration>
```

## 解决方式
1. 出现上述问题的原因是因为MyBatis输出SQL语句的来源不是 `org.mybatis` 
2. 而是发起查询的接口，这种接口一般都是项目自定义的包，例如 `com.asing1elife` 
3. 所以需要在 *logback.xml* 中将自定义包的日志级别设定为 *DEBUG* ，例如 `<logger name="com.asing1elife" level="DEBUG" />`