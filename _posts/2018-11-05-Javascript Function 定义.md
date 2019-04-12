---
title: Javascript Function 定义
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数基础语法和定义  

## 语法
2. proName1 ... proNameN 即参数列表，直接写形参名称即可，不用加 var ，参数列表并不是必须的
3. 参数列表相当于函数的入口，return 语句相当于函数的出口

```js
function [funcName]（proName1, proName2, ... , proNameN） {
	// 执行部分
	var result = proName1 + proName2 + ... + proNameN
	return result		// 函数不需要指定返回值类型，但可以有返回值
}
```

## 函数也是一种数据类型，其类型为 function
```js
function test() {
	console.log('test')
}

alert(typeof test)		// 显示 function 
```

## 函数可以被当做一种数据类型进行传递
```js
function test1(example) {
	example()	
}
	
function test2() {
	alert('执行方法')
}
	
// 将 test2 方法作为参数传递到 test1 中
test1(test2)		// 显示 执行方法
```

## 函数名称并不是必须的，不带函数名称的函数称为匿名函数
```js
alert(function() { alert('INTO METHOD') })		// 显示 INTO METHOD
```

## 函数可以进行嵌套，但内部函数的作用域只在其外部函数之内才有效
1. 此方法不推荐使用，不具备实际意义

```js
function test3() {
	// 定义方法
	function test4() {
		alert('INTO METHOD test4')	
	}	

	test4()		// 执行方法
}
	
// 调用 test3 方法
test3()		// 显示 INTO METHOD test4
```