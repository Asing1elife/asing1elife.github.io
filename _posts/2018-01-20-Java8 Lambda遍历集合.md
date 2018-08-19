---
title: Java8 Lambda遍历集合
date: 2018-01-20
author: asing1elife
categories:
 - java
 - jdk8
tags:
 - java
 - jdk8
 - for
 - map
---
> Java8支持使用Lambda语法对集合进行遍历，语法如下  

## List
```java
allQuestions.forEach(question -> {
    if (question.getType().getCode().equals(typeCode)) {
        filterQuestions.add(question);
    }
});
```

## Map
```java
items.forEach((k, v) -> {
    System.out.println("Item : " + k + " Count : " + v);
    if("E".equals(k)) {
        System.out.println("Hello E");
    }
});
```