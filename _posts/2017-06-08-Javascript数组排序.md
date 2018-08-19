---
title: Javascript数组排序
date: 2017-06-08
author: asing1elife
categories:
 - javascript
 - array
tags:
 - javascript
 - array
---
> Javascript 提供 sort() 函数用于排序  
> 但默认情况是使用字符串 Unicode 码点排序  

## 问题
```javascript
// 得到的结果是 [1, 10, 2, 5]
[1, 5, 2, 10].sort();
```

## 解决
```javascript
[1, 5, 2, 10].sort((a, b) => a - b);
```