---
title: MyBatis的settings配置表
date: 2017-03-22
author: asing1elife
categories:
 - java
 - mybatis
tags:
 - java
 - mybatis
---
> `<settings/>` 标签是 MyBatis 中的属性配置项，可以改变 MyBatis 的运行时行为  

## 规则 
> **标签中的 value 为该设置的默认值**

1. 缓存全局开关
	* `<setting name="cacheEnabled" value="true" />`
2. 延迟加载全局开关
	* 值为 true 时所有对象延迟加载
	* 可通过在具体的查询中设置 fetchType 来覆盖该设置
	* `<setting name="lazyLoadingEnabled" value="false" />`
3. 所有对象直接加载
	* `<setting name="aggressiveLazyLoading" value="true" />`
4. 允许单一语句返回多个结果集，需要兼容驱动
	* `<setting name="multipleResultSetsEnabled" value="true" />`
5. 使用列标签代替类名
	* `<setting name="useColumnLabel" value="true" />`
6. 允许 JDBC 使用数据库自增主键
	* `<setting name="useGeneratedKeys" value="true" />`
7. 指定自动映射到字段的规则
	* NONE 取消自动映射
	* PARTIAL 只映射没有定义嵌套结果集映射的结果集
	* FULL 自动映射任何结果集
	* `<setting name="autoMappingBehavior" value="PARTIAL" />`
8. 指定当自动映射碰到未知列的处理规则
	* NONE 不做任何处理
	* WARNING 输入警告日志
	* FAILING 抛出 SqlSessionException 异常
	* `<setting name="autoMappingUnknownColumnBehavior" value="NONE" />`
9. 配置默认执行器
	* SIMPLE 普通执行器
	* REUSE 重用预处理语句
	* BATCH 重用语句并执行批量更新
	* `<setting name="defaultExecutorType" value="SIMPLE" />`
10. 设置驱动等待数据库响应的超时时间
	* 该设置项默认没有值
	* 值的范围是**任意正整数**
	* 值的单位是**秒**
	* `<setting name="defaultStatementTimeout" value="" />`
11. 为驱动的结果集数量设置提示值
	* 该设置项默认没有值
	* 值的范围是**任意正整数**
	* 可在具体查询中通过 fetchSize 覆盖该设置项
	* `<setting name="defaultFetchSize" value="" />`
12. 允许在嵌套语句中使用分页 RowBounds
	* `<setting name="safeRowBoundsEnabled" value="false" />`
13. 允许在嵌套语句中使用分页 RowHandler
	* `<setting name="safeRowHandlerEnabled" value="true" />`
14. 开启驼峰命令规则自动转换功能
	* 例如：create_time > createTime
	* `<setting name="mapUnderscoreToCameCase" value="false" />`
15. 利用本地缓存机制防止循环引用和加速重复嵌套查询
	* SESSION 缓存一个会话中执行的所有查询
	* STATEMENT 本地会员只用在语句执行中，对相同 SqlSession 的不同调用不会共享数据
	* `<setting name="localCacheScope" value="SESSION" />`
16. 当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型
	* OTHER 一般类型
	* NULL 空值
	* VARCHAR 字符串
	* `<setting name="jdbcTypeForNull" value="OTHER" />`
17. 指定某个对象的方法触发一次延迟加载
	* 多个方法名称通过逗号划分
	* `<setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString />"`
18. 指定动态 SQL 生成的默认语言
	* `<setting name="defaultScriptingLanguage" value="org.apache.ibatis.scripting.xmltags.XMLLanguageDriver" />`
19. 当结果集为 NULL 时，调用映射对象的 setter 方法为结果赋值
	* 结果集类型是 List 时，调用 setter 方法
	* 结果集类型是 Map 时，调用 put 方法
	* 该属性对于基本类型无效
	* `<setting name="callSetterOnNulls" value="false" />`
20. 当对象所有列都返回 NULL 时，将整个对象设置为 NULL
	* `<setting name="returnInstanceForEmptyRow" value="false" />`
21. 指定日志名称的前缀
	* 该设置项没有默认值
	* 值可以是**任何字符串**
	* `<setting name="logPrefix" value="" />`
22. 指定日志的具体实现方式
	* 该设置项没有默认值
	* 值可以是 slf4j / log4j / log4j2 / jdk_logging / commons_logging / stdout_logging / no_loggging
	* 未指定值的时候会在上述支持列表中自动查找
	* `<setting name="logImpl" value="" />`
23. 指定创建具有延迟加载能力的对象所用到的代理工具
	* 值可以是 CGLIB / JAVASSIST
	* `<setting name="proxyFactory" value="JAVASSIST" />`
24. 指定 VFS 的实现
	* 该设置项没有默认值
	* 值可以是自定义 VFS 的实现类全名
	* 多个 VFS 可以通过逗号划分
	* `<setting name="vfsImpl" value="" />`
25. 允许使用方法签名中的名称作为语句参数名称
	* 仅在 Java 8 环境中生效，并且需要在环境变量中加上 -parameters 
	* `<setting name="useActualParamName" value="true" />`

## 完整配置示例  
```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```