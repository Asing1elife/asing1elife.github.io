---
title: jQuery exist 自定义方法
date: 2016-12-28
author: asing1elife
categories:
 - jquery
tags:
 - jquery
---
> 一般通过 `$(“#element”).length === 1` 判断元素是否存在，但这样显得过于麻烦  

## 解决
1. 可集成 length 方法，扩展成为 exist 方法
```js
jQuery.fn.exists = function() {
	return this.length > 0;
}
```