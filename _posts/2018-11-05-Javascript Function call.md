---
title: Javascript Function call 
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数中 call() 的使用  

## call & apply
1. 每一个函数都包含上述两个非继承而来的方法
2. 用法：绑定一些参数，用于传递参数及调用

```js
function sum(x, y) {
	return x + y
}

function callSum(num1, num2) {
	// 将 num1 num2 作为参数绑定到以 sum 为当前作用域的函数中
	return sum.call(this, num1, num2)
}

function applySum(num1, num2) {
	// 将 num1 num2 作为数组元素绑定到以 sum 为当前作用域的函数中
	return sum.apply(this, [num1, num2])	
}

/**
 * 其实直接调用 sum 并传递值即可
 * 因此上述两种普通用法并不实用
 */
function normalSum(num1, num2) {		
	return sum(num1, num2)	
}

alert(callSum(10, 20))		// 显示 30
alert(applySum(20, 30))		// 显示 50
alert(normalSum(30, 40))		// 显示 70
```

## 扩充函数赖以运行的作用域
```js
window.color = 'red'
var obj = {color : 'blue'}

function showColor() {
	alert(this.color)	
}

/**
 * 根据 call(value) 中 value 传递的作用域不同
 * showColor() 方法中的 this.color 访问的参数也不同 
 *
 * 关键作用：利用此方法可以让函数获得最大限度的重用性
 * 让同一个函数可以再不同的作用域下实现不同的效果
 *
 * 优势：被调用函数所使用的对象不需要和当前函数存在任何耦合关系
 * 最大限度的增加了灵活性
 */
showColor.call(window)		// 显示 red
showColor.call(this)		// 显示 red
showColor.call(obj)		// 显示 blue
```

## call 方法的简单模拟
```js
function test(num1, num2) {
	return num1 + num2
}

// 在 JS 中，当函数名大写，则被认定为是 自定义对象
function Obj(x, y) {
	this.x = x
	this.y = y
}

var o = new Obj(10, 20)		// 新建一个 Obj 对象，并传入两个参数

// 通过调用 call() 方法，将函数 test 的作用域扩展到对象 Obj 中
// 相当于让对象 Obj 临时拥有 test 函数
alert(test.call(o, o.x, o.y))		// 显示 30

// 上述 call() 方法的调用实际上是通过以下三步实现
o.method = test		// 先将 test 函数临时指定给对象 o
alert(o.method(o.x, o.y))		// 再通过 o.method 传递参数到方法内部，并显示
delete o.method		// 最后删掉该临时方法
```