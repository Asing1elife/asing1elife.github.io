---
title: Maven打包时抛出should not point at files within the project directory …异常
date: 2018-05-09
author: asing1elife
categories:
 - java
 - maven
 - error
tags:
 - java
 - maven
 - error
---
> 项目中通过 Maven 引入本地包后打包时抛出 should not point at files within the project directory … 警告  

## 具体问题
1. 在项目中引入本地包

```xml
<dependency>
    <groupId>ppts.model</groupId>
    <artifactId>ppts-model</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>system</scope>
    <systemPath>${basedir}/../lib/ppts-model-1.0-SNAPSHOT.jar</systemPath>
</dependency>
```

2. 在项目进行 package 打包时，抛出以下异常，且并没有引入对应 jar 包

```java
[WARNING] 'dependencies.dependency.systemPath' for ts.core:ts-core:jar should not point at files within the project directory, ${basedir}/../lib/ts-core-1.0.jar will be unresolvable by dependent projects @ line 97, column 19
```

## 解决方式
1. 移除本地包依赖中的 `<scope/>` 和 `<systemPath/>` 

```xml
<dependency>
  <groupId>ppts.model</groupId>
  <artifactId>ppts-model</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

2. 通过 **maven-install-plugin** 插件对 jar 包进行安装
    * `<phase>clean</phase>` 表示该 jar 包会在执行 clean 操作时引入

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-install-plugin</artifactId>
  <version>2.5.2</version>
  <executions>
    <execution>
      <id>install-ppts-model</id>
      <phase>clean</phase>
      <configuration>
        <file>${basedir}/../lib/ppts-model-1.0-SNAPSHOT.jar</file>
        <repositoryLayout>default</repositoryLayout>
        <groupId>ppts.model</groupId>
        <artifactId>ppts-model</artifactId>
        <version>1.0-SNAPSHOT</version>
        <packaging>jar</packaging>
        <generatePom>true</generatePom>
      </configuration>
      <goals>
        <goal>install-file</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```