---
title: Javascript 为 String 提供分离字符的全局方法
date: 2017-08-20
author: asing1elife
categories:
 - javascript
tags:
 - javascript
---
> 在一个字符串的每个字符之间插入空格，并且这个方法可以直接被字符串调用  

## 实现方式
```javascript
String.prototype.spacify = function(){  
  return this.split("").join(" ");
};
```