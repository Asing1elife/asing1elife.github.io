---
title: Javascript Object
date: 2018-11-05
author: asing1elife
categories:
 - javascript
tags:
 - javascript
---
> 所有类的基础类  

## 语法
```js
var obj = new Object();		// 创建一个空的 Object 对象
var obj = {};		// 实例化的简便写法
```

### 给对象设置属性，不需要提前声明
```js
obj.name = 'Jone';
obj.age = 20;
obj['sex'] = 'Male';
	
```

### 给对象设置方法，方法名可以随意指定
```js
obj.say = function() {
	alert('Hello World !!');
}
	
```

### 访问对象的属性和方法
```js
alert(obj.name);		// 显示 Jone
obj.say();		// 显示 Hello World !! 方法名后需要跟一对小括号
	
```

### 删除对象的属性和方法
```js
delete obj.name;		// 删除属性
delete obj.say;		// 删除方法，不能带小括号，否则会被视为调用此方法
alert(obj.name);		// 显示 undefined
obj.say();		// Console 提示方法未定义
```

## for (var attr in object) { ... } 
1. 遍历对象的属性和方法的名称
2. attr 当前属性和方法的名称
3. object 指定对象

```js
for (var attribute in obj) {
	// 遍历对象的所有属性和方法的名称并显示
	alert(attribute);
	// 遍历对象的所有属性和方法的内容
	alert(obj[attribute]);
}
```

## constructor 
1. 保存对象的创建函数

```js
alert(obj.constructor);		// 显示 obj 的创建函数 function Object() { [native code] }
```

## hasOwnProperty(propertyName)
1. 检测传入的参数名称是否存在于指定对象中

```js
var result = obj.hasOwnProperty('name');		// 检测 name 属性是否存在于 obj 对象中，返回 true 或 false
alert(result);		// 返回 true ，说明 obj 中存在 name 属性
```

## propertyIsEnumerable(propertyName)
1. 检测传入的参数是否可以被 for-in 枚举循环遍历

```js
var result = obj.propertyIsEnumerable('name');		// 检测 name 属性是否可以被 for-in 遍历，返回 true 或 false
alert(result);		// 返回 true ，说明 name 属性可以被 for-in 遍历
```

## toLocaleString()
1. 返回指定对象的字符串表示，该字符串与执行环境相对应

```js
var result = obj.toLocaleString();
alert(result);		// 显示 [object Object]
```

## toString()
1. 返回指定对象的字符串表示

```js
var result = obj.toString();
alert(result);		// 显示 [object Object]
```

## valueOf()
1. 返回对象的字符串、数值或布尔值表示，并无强制类型

```js	 
var result = obj.valueOf();
alert(result);		// 显示 [object Object]
```