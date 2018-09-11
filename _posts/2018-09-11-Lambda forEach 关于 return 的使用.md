---
title: Lambda forEach 关于 return 的使用
date: 2018-09-11
author: asing1elife
categories:
 - java
 - lambda
tags:
 - java
 - lambda
---
> JDK8 中新增的 Lambda 表达式对于 for 循环的操作变得非常简洁  
> 但其中的 forEach 和 for 之间存在一定差异  
> 比如 forEach 无法使用 break 和 continue  

## forEach 实现和 contiune 一样的效果
1. 参见以下代码可知，在 **forEach** 中 `return` 可实现和 `contiune` 一样的效果

```java
int[] arrs = new int[]{1, 3, 9, 2};

arrs.forEach(arr -> {
	if (arr > 4) {
		return;
	}

	// 输出 1 3 2
	System.out.println(arr);
})
```

## forEach 实现和 break 一样的效果
1. 对不起，臣妾做不到