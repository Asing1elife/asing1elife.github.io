---
title: Javascript Function 执行环境
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数的执行环境  

## 执行环境（execution context）
1. 定义了变量或函数有权访问的其他数据，决定了各自的行为
2. 每一个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中
3. 全局执行环境是最外围的一个执行环境
4. 根据 JS 所在的宿主环境（window 对象为最上层的执行环境）不同，表示执行环境的对象也不一样
5. 执行环境是抽象存在的，代码无法访问

```js
var a = 10
var b = 20
function cc() {
	// do something	
}

var Obj()		// 执行环境的含义相当于上述变量 a b 以及函数 cc 都保存在 Obj 对象中
```

## 作用域链（scope chain）
1. 保证对执行环境有权访问的所有变量和函数的有序访问
2. 执行环境的特点是层层追溯，并形成一个作用域链

```js
var color1 = 'red'		// 存在于一级作用域中

function showColor() {
	var color2 = 'blue'		// 存在于二级作用域中
	
	function swapColor() {
		var color3 = color2		// 存在于三级作用域中
		color2 = color1
		color1 = color3	
	}
	// 只能访问到 color1 color2 
	swapColor()	
}
// 只能访问到 color1
showColor()
```