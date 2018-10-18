---
title: Javascript数组去重
date: 2017-01-09
author: asing1elife
categories:
 - javascript
 - array
tags:
 - javascript
 - array
---
> Set是ES2015引入的数据类型，意为集合  
> 其不允许重复元素出现的特性，对于NaN、undefined、null都适用  

## 实现方式
```javascript
function unique(arr) {
	var set = new Set(arr);
	
	return Array.from(set);
}


var arr = [1,1,'1','1',0,0,'0','0',undefined,undefined,null,null,NaN,NaN,{},{},[],[],/a/,/a/];
console.log(unique(arr));
```