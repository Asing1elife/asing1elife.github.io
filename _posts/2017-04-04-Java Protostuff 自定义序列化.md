---
title: Java Protostuff 自定义序列化
date: 2017-04-04
author: asing1elife
categories:
 - java
tags:
 - java
 - serializable
---
> 虽然 Java 提供内置的序列化 API **Serializable** ，但其效率并不是最高的  
> Google 提供了一个效率很高的序列化 API **Protobuf** ，但其使用过于复杂  
> 开源社区在 Protobuf 的基础上封装出 **Protostuff** ，在不丢失效率的前提上，让使用步骤变得更简单  
> 一般情况下 Protostuff 序列化后的数据大小是 Serializable 的 1/10 之一，速度更是两个量级以上  

## 依赖
```xml
<dependency>
	<groupId>com.dyuproject.protostuff</groupId>
	<artifactId>protostuff-core</artifactId>
	<version>1.1.1</version>
	</dependency>
<dependency>
	<groupId>com.dyuproject.protostuff</groupId>
	<artifactId>protostuff-runtime</artifactId>
	<version>1.1.1</version>
</dependency>
```


## 创建 Schema 序列化规则对象
1. 规则对象只接受任何带 set/get 方法的 POJO 对象

``` java
private RuntimeSchema<RecordDTO> schema = RuntimeSchema.createFrom(RecordDTO.class);
```

## 数据序列化
```java
public void setRecord(RecordDTO record) {
	// 将未序列化的对象数据通过 schema 规则进行序列化
	// LinkedBuffer 缓存器的作用是当对象数据过大时，可以对数据进行缓存，从而实现分布序列化
	byte[] bytes = ProtobufIOUtil.toByteArray(record, schema, LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE));
}
```

## 数据反序列化
```java
public RecordDTO getRecord(byte[] bytes) {
	// 空的对象数据
	RecordDTO record = schema.newMessage();

	// 将被序列化的对象数据通过 schema 规则反序列化转换到空的对象数据中
	ProtobufIOUtil.mergeFrom(bytes, record, schema);

	return record;
}
```