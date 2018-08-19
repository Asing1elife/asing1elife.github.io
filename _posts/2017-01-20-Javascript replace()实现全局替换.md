---
title: Javascript replace()实现全局替换
date: 2017-01-20
author: asing1elife
categories:
 - javascript
tags:
 - javascript
---
> 第一个参数需要用到正则表达式，`/g` 表示全局替换，否则只能替换第一个匹配项  

## 实现方式
```javascript
skill.replace(/a/g, "A");
```