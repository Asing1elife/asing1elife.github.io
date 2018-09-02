---
title: MySQL DATE_ADD() 方法
date: 2018-03-22
author: asing1elife
categories:
 - database
 - mysql
tags:
 - database
 - mysql
---
> 用于向日期添加指定的时间间隔  

## 语法
```sql
DATE_ADD(date, INTERVAL num TYPE)
```

## 示例
```sql
DATE_ADD('2018-03-22 10:23:00', INTERVAL 30 MINUTE)
``` 

## 解析
1. `date` 只要是合法的日期表达式即可
2. `num` 是希望添加的时间间隔值
3. `TYPE` 是时间间隔的单位
	* MICROSECOND
	* SECOND
	* MINUTE
	* HOUR
	* DAY
	* WEEK
	* MONTH
	* QUARTER
	* YEAR
	* SECOND_MICROSECOND
	* MINUTE_MICROSECOND
	* MINUTE_SECOND
	* HOUR_MICROSECOND
	* HOUR_SECOND
	* HOUR_MINUTE
	* DAY_MICROSECOND
	* DAY_SECOND
	* DAY_MINUTE
	* DAY_HOUR
	* YEAR_MONTH