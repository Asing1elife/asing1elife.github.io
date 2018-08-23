---
title: jQuery one() 防止重复点击
date: 2017-04-02
author: asing1elife
categories:
 - jquery
tags:
 - jquery
---
> $.one() 方法的意思是给指定按钮绑定一次性事件，即使用一次便失效  

## 实现方式
```js
$(".submit-btn").one("click", function () {
	...
});
```