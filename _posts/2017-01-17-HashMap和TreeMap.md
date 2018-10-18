---
title: HashMap和TreeMap
date: 2017-01-17
author: asing1elife
categories:
 - java
tags:
 - java
 - map
---
> 简单描述一下HashMap和TreeMap的区别

## 介绍
1. Map是key-value的集合接口，其实现类包括：
	1. HashMap - 值没有顺序
	2. TreeMap - key值默认升序
	3. LinkedHashMap - 值没有顺序

## key排序
```java
Map<String, String> map = new TreeMap<String, String>(
	new Comparator<String>() {
		public int compare(String obj1, String obj2) {
			// 降序排序
			return obj2.compareTo(obj1);
		}
	}
);
```	

## value排序
```java
// 这里将map.entrySet()转换成list
List<Map.Entry<String,String>> list = new ArrayList<Map.Entry<String,String>>(map.entrySet());
	// 然后通过比较器来实现排序
	Collections.sort(list,new Comparator<Map.Entry<String,String>>() {
		// 升序排序
		public int compare(Entry<String, String> o1, Entry<String, String> o2) {
			return o1.getValue().compareTo(o2.getValue());
		}            
	}
);
```
