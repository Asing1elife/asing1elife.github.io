---
title: var let const 的区别
date: 2017-11-23
author: asing1elife
categories:
 - javascript
 - es6
tags:
 - javascript
 - es6
---

> var是ES6之前JavaScript用于定义变量的语法，而let const是ES6之后JavaScript用于定义变量的语法  

## var存在的两个bug
1. JS没有块级作用域
	* 通过var声明的变量，其作用域是函数的全部
2. 循环内变量会过度共享
	* 意思就是说在循环内部定义的变量，在循环外部依旧可以访问

## let存在的意义
1. let声明的变量拥有块级作用域
	* let声明的变量其作用域只是外层块，而不是外层函数
2. let声明的全局变量不是全局对象的属性
	* 通过let声明的全局变量无法通过window.变量名进行访问，其只存在于一个相对的作用域中
3. 形如 `for(let x in data)` 的循环在每次迭代时都会为 `x` 创建新的绑定
4. let声明的变量无法重新被定义
5. let变量在声明前无法被调用
	* 这个比较基于花括号包裹的作用域中声明了变量name ，所以JS编译器会根据ES6规定的 **变量在声明之前无法使用** 抛出错误
	* var声明的变量可以被使用是因为存在 **变量提升**
```js
if (true) {
  name = 'abc' // name is no defined
  let name
}

if (true) {
  name = 'abc'
  console.log(name) // abc
  var name
}
```

## const的作用
1. const就是用于定义常量的，常量（不会改变的变量）有以下特点
	* 声明时必须赋值
	* 赋值后不可以改变