---
title: Javascript Function 性质
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数的性质以及 arguments 使用  

## 参数性质
1. 上述实例说明在 JS 中，形参个数并不需要和实参个数相对应

```js
function test(a, b, c, d) {
	return a + b	
}
alert(test(10, 20))		// 显示 30
```

## arguments 使用
1. arguments 是函数内部用于接收实际参数的数组
2. 该数组只能在函数内部被访问

```js
function test(a, b, c, d) {
	var formal = test.length		// 获取函数的形参个数
	alert(formal)		// 显示 4
	
	var actual = arguments.length		// 获取函数的实参个数	
	alert(actual)		// 显示 2
	
	var parameter = arguments[0]		// 获取数组中的第一个实参值
	alert(parameter)		// 显示 10
	
	// 判断 test 中形参和实参的个数是否相同
	if (test.length == arguments.length) {
		return a + b
	} else {
		return '参数个数不符'
	}
}
alert(test(10, 20))		// 显示 参数个数不符
```

## arguments 扩展
1. arguments.callee 表示对所属函数自身的调用，多用于递归

```js
function fact(num) {
	if (num <= 1) {
		return 1
	} else {
		return num * arguments.callee(num - 1)	
	}
}
alert(fact(5))		// 显示 120 ，该运算规则为 5 的阶乘，相当于 5 * 4 * 3 * 2 * 1
```