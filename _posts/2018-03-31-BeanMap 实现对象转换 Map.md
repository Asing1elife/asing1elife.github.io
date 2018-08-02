---
title: BeanMap 实现对象转换 Map
date: 2018-03-31
author: asing1elife
categories:
 - java
 - map
tags:
 - java
 - map
 - spring
---
> 通过 **org.springframework.cglib.beans** 的 BeanMap 可以实现对象将字段和字段值直接转换为 Map 的 key-value 形式  

## 实现方式
```java
SortedMap<String, String> signMaps = Maps.newTreeMap();

BeanMap beanMap = BeanMap.create(ext);

for (Object key : beanMap.keySet()) {
    Object value = beanMap.get(key);

    // 排除空数据
    if (value == null) {
        continue;
    }

    signMaps.put(key + "", String.valueOf(value));
}
```