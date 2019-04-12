---
title: Javascript 数组去重
date: 2017-01-09
author: asing1elife
categories:
 - javascript
 - array
tags:
 - javascript
 - array
---
> 数组去重的多种方式  

## Set 方式
1. Set是ES2015引入的数据类型，意为集合
2. 其不允许重复元素出现的特性，对于NaN、undefined、null都适用

```javascript
function unique(arr) {
	var set = new Set(arr);
	
	return Array.from(set);
}


var arr = [1,1,'1','1',0,0,'0','0',undefined,undefined,null,null,NaN,NaN,{},{},[],[],/a/,/a/];
console.log(unique(arr));
```

## Object 方式
1. 在 JS 对象中，key 永远不会重复

```js
var arr = [1, 2, 1, 3, 4, 6, 3, 4, 7];
	
// 将数组转换为对象
function toObject(arr) {
	var obj = {};		// 实例化一个空对象
	// 遍历数组的同时为 obj 赋值，把数组中的值转变为 obj 的 key
	arr.forEach(function(item, index, array) {
		obj[item] = item;		// 为 obj 绑定 key-value 关系
	});
	
	return obj;		// 返回转换后的对象
}
	
 // 将对象转换为数组
 function toArray(obj) {
 	var arr = [];		// 实例化一个空数组
 	// 遍历对象的同时为 arr 赋值
 	for (var attr in obj) {
 		// 判断当前 attr 是否属于 obj
 		if (obj.hasOwnProperty(attr)) {
 			arr.push(attr);		// 将当前 attr 传入 arr 中
 		}
 	}
 	
 	return arr;		// 返回转换后的数组
}
	
// 去除数组中的重复项
function toShow(arr) {
	// 1. 先将数组转换为对象
	// 2. 再将对象转换为数组
	return toArray(toObject(arr));		// 依次调用两个方法
}

alert(toShow(arr));		// 显示 1,2,3,4,6,7
```