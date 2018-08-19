---
title: Javascript 跳出循环
date: 2017-01-20
author: asing1elife
categories:
 - javascript
tags:
 - javascript
 - for
---
> 对于不同的循环方式，javascript需要采用不同的方式跳出  

## for循环依然使用continue和break
```javascript
for (var i = 0; i < length; i++) {
	if (i === 1) {
		break;
	} else {
		continue;
	}
}
```

## each循环则使用return true和return false
```javascript
$.each(arr, function (index, value) {
	if (index == 1) {
		return false;
	} else {
		return true;
	}
});
```