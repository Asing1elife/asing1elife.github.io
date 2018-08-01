---
title: SpringMVC的CacheManager配置
date: 2017-03-27
author: asing1elife
categories:
 - java
 - springmvc
tags:
 - java
 - springmvc
 - cache
---
> Spring 3.1 后提供一个新特性 **基于注释驱动的缓存**  
> 可以通过在方法上加入注解，从而缓存该方法返回的数据  

## 编写配置文件
```xml
<!-- 使用缺省名称为 cacheManager 的缓存管理器，其缺省实现为 org.springframework.cache.support.SimpleCacheManager -->
<cache:annotation-driven/>

<!-- 配置 cacheManager -->
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
<!-- 配置缓存集合 -->
<property name="caches">
<set>
<!-- 缺省方案 -->
<bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default"/>
<!-- 自定义方案 -->
<bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="dictionaryCache"/>
</set>
</property>
</bean>

```

## 在需要缓存的方法上加缓存注释，并指定自定义名称
```java
@Cacheable(value = "dictionaryCache")
public List<DictionaryDTO> getDictionaries(String className) {
return dictionaryDao.getDictionaries(className);
}
```

## 清空缓存
* `allEntries = true` 表示清空所有缓存
* `key = "#dictionary.getName()"` 表示只清空方法参数中带有指定key的缓存

```java
@CacheEvict(value = "departmentService", allEntries = true)
public ResponseData removeDictionary(Long id) {
...
}

@CacheEvict(value = "departmentService", key = "#dictionary.getName()")
public void addDictionary(DictionaryDTO dictionary) {
...
}
```