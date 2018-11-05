---
title: Javascript Function 声明
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数的三种声明方式  

## 三种定义函数的方式
### 语句形式
```js
function test1() {
	alert('INTO METHOD test1')	
}
test1()		// 显示 INTO METHOD test1
```

### 直接量形式
```js
// 将一个匿名函数指定个一个变量
var test2 = function() {
	alert('INTO METHOD test2')
}
test2()		// 显示 INTO METHOD test2
```

### 构造函数形式 - 顶级作用域
```js
// 将一个匿名函数的构造函数指定给一个变量
// 在匿名函数的参数列表中规定了参数以及返回值规则 
var test3 = new Function('a', 'b', 'return a + b')
alert(test3(10, 20))		// 显示 30
```

## 语句形式和构造函数形式的效率比较
1. 上述两个实例的比较说 **语句形式** 效率要比 **构造函数** 形式高很多
2. 因为 **语句形式** 在被解析器解析后便存放于内存中，不会再次被创建，是一种静态创建方法的形式
3. 而 **构造函数** 形式则会在每次访问时被解析器创建，是一种动态创建方法的形式
4. 从而表现出来的差异在于，**语句形式** 虽然占用内存，但效率更高

```js
var date1 = new Date()
var time1 = date1.getTime()
for (i = 0 i < 100000 i++) {
	function test() {}		// 创建一个 function 语句形式的空函数
}

var date2 = new Date()
var time2 = date2.getTime()
alert(time2 - time1)		// 显示 1-3 之间，说明执行 100000 次 test 函数所花费的时间在 1-3 毫秒之间
	
var date1 = new Date()
var time1 = date1.getTime()
for (i = 0 i < 100000 i++) {
	var test = new Function()		// 创建一个 function 构造函数形式的空函数
}

var date2 = new Date()
var time2 = date2.getTime()
alert(time2 - time1)		// 显示 250-270 之间，说明执行 100000 次 test 函数所花费的时间在 250 - 270 毫秒之间
```

## 语句形式和直接量形式的解析顺序比较
1. 上述两个实例的比较说明 **语句形式** 相对于 **直接量形式** 用着更加优先解析顺序
2. 对于 **语句形式** ，JS 解析器会优先解析，当确定页面中没有 function 语句形式的代码后，才会按顺序执行其他代码
3. 对于 **直接量形式** ，JS 解析器虽然会优先解析其直接量，但是该方法的实例化则需要等到代码顺序执行到该行

```js
test1()		// 显示 INTO MEHOTD test1 ，说明在 test1 方法被调用之前，已经被解析器创建
function test1() {
	alert('INTO METHOD test1')
}

test2()		// Console 报错：test2 is not a function ，说明低于解析器来说，test2 这个方法还没被创建
var test2 = function() {
	alert('INTO METHOD test2')	
}
```

## 三种形式综合比较
```js
function f() { return 1 }
alert(f())		// 显示 4 ，说明 JS 解析器优先创建了 f 的语句形式，最终 f1 被 f4 覆盖

var f = new Function('return 2')
alert(f())		// 显示 2 ，说明 f4 被 f2 覆盖

var f = function() { return 3 }
alert(f())		// 显示 3 ，说明 f2 被 f3 覆盖

function f() { return 4 }
alert(f())		// 显示 3 ，说明 f4 在被 JS 解析器优先创建后，就不再创建，所以 f3 依旧有效 

var f = new Function('return 5')
alert(f())		// 显示 5 ，说明 f3 被 f5 覆盖

var f = function() { return 6 }
alert(f())		// 显示 6 ，说明 f5 被 f6 覆盖
```

## 三种形式作用域比较
1. 上述实例说明，function 构造函数形式拥有独特的顶级作用域
2. 而 function 的语句形式同 function 直接量形式只有函数作用域

```js
var k = 1		// 全局
function test1() {
	var k = 2		// 局部
	function test() { return k }		// function 语句形式
	test()
}
test1()		// 显示 2 ，说明 function 语句形式是函数作用域，只能访问到其上层方法以内的局部变量 k
	
var k = 1		
function test1() {
	var k = 2		
	var test = function() { return k }		// function 直接量形式	
	test()
}
test1()		// 显示 2 ，说明 function 的直接量形式作用域同 function 语句形式相同

var k = 1
function test1() {
	var k = 2
	var test = new Function('return k')
	test()	
}
test1()		// 显示 1 ，说明 function 构造函数虽然在 test1 中被创建，但实际上其拥有顶级作用域，相当于是在 test1 之外被创建
```