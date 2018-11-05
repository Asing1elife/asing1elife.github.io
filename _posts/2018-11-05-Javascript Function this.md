---
title: Javascript Function this
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - function
tags:
 - javascript
 - function
---
> 讲解函数中 this 的使用  

## 定义
1. this 对象在运行时基于函数的执行环境绑定
2. 通俗的将就是说 this 对象总是指调用者
3. this 说“谁调用了我，我就只想谁”
4. 在全局环境中，this 等于 window
5. 在对象方法调用中，this 等于该对象

```js
var k = 10
function test() {
	this.k = 20	
}

test()		// 规范写法 window.test()
alert(test.k)		// 显示 undefined

/**
 * alert() 调用 test() 中的 k ，但其实 test() 中并不存在一个局部变量 k ，所以显示 undefined
 * 当在 test() 中通过 this 为变量 k 赋值时，实际上是为在 window 中的变量 k 赋值
 * 因为 this 指向的是调用者，test() 方法的调用者是 window ，所以 this 指向的调用者也是 window
 */
 
alert(window.k) or alert(k)		// 显示 20
```