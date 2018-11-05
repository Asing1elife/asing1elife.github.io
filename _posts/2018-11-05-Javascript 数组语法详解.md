---
title: Javascript 数组语法详解
date: 2018-11-05
author: asing1elife
categories:
 - javascript
 - array
tags:
 - javascript
 - array
---
> JavaScript 中的数组相当于 Java 中的 Map ，数组本身是一个对象，属于引用数据类型  

## 定义
1. 既然数组是一个对象，当进行 alert(arr) 时
2. 应该输出 [object object] ，可是却输出的是数组中具体的值
3. 因为 alert() 会对将要输出的值进行隐式的 toString() 转换
4. 通过转换，数组 arr 便直接输出其内部值

```js
var arr = new Array(6)		// 甚至不用预先声明，不用 new
var arr = [1, 2, 3, 'aa', new Date(), true]		// 元素类型可任意选择
alert(arr)		// 显示 1,2,3,4,aa,true
arr.length = 5
alert(arr)		// 显示 1,2,3,4,aa ，由于数组长度的改变，大于 5 的部分被自动舍弃，长度可以随意改变
```

## push
1. 往指定数组尾部推送值，并返回值

```js
var arr = []
arr.push(1)		
arr.push(2, 3)	// 可以是单个值，也可以是多个值
var result = arr.push(4, 5)		
alert(arr)		// 显示 1,2,3,4,5 表示通过 push() 推送到数组中的值不会被覆盖
alert(result)		// 显示 5 ，push() 在推送值的同时，可以返回数组的当前长度
```

## pop
1. 从指定数组尾部移除一个元素，并返回值

```js
var arr = [1, 2, 3, 4, 'aa']
var result = arr.pop()		
alert(arr)		// 显示 1,2,3,4
alert(result)		// 显示 aa ，pop() 在移除元素的同时会将被移除的值作为返回值 
```

## shift
1. 从指定数组头部移除一个元素，并返回值

```js
var arr = ['aa', 1, 2, 3, 4]
var result = arr.shift()
alert(arr)		// 显示 1,2,3,4
alert(result)		// 显示 aa ，shift() 在移除元素的同时会将移除的值作为返回值
```

## unshift
1. 往指定数组头部推送值，并返回值

```js
var arr = [1, 2, 3, 4]
var result = arr.unshift('aa', 'bb')
alert(arr)		// 显示 aa,bb,1,2,3,4
alert(result)		// 显示 6 ，unshift() 在推送值的同时，可以返回数组的当前长度
```

## splice(startIndex, spliceLength, var1, var2, ... varN)
1. 截取字符串的同时，拼接新字符串到指定位置
2. startIndex 截取的起始位置
3. spliceLength 截取的长度
4. var1, var2, ... varN 待拼接的新字符串

```js
var arr = [1, 2, 3, 4, 5]
arr.splice(1, 2, 3, 4, 5)
alert(arr)		// 显示 1,3,4,5,4,5 从数组 arr 的索引 1 开始，截取长度为 2 ，将 [2, 3] 替换为 [3, 4, 5]
	
var arr1 = [1, 2, 3, 4, 5]
arr1.splice(1, 2)
alert(arr1)		// 显示 1,4,5 从数组 arr1 的索引 1 开始，截取长度为 2 ，且无带拼接字符串

var arr2 = [1, 2, 3, 4, 5]
arr2.splice(1)
alert(arr2)		// 显示 1 ，从数组 arr2 的所以 1 开始，在未指定长度的情况下，后续字符串全部截断
```

## slice
1. 截取字符串
2. 不操作数组本身，而是返回被截取的内容

```js
var arr = [1, 2, 3, 4, 5]
var result = arr.slice(2, 4)		// 采用左闭右开区间 [2, 4) 也就是索引从第 2 位开始，到第 4 位结束，且不包括第 4 位
alert(result)		// 显示 3,4
```

## concat
1. 在指定数组后连接新的数组或值
2. 不操作数组本身，而是返回连接后的结果

```js
var arr1 = [1, 2, 3]
var arr2 = [4, 5]
var result = arr1.concat(arr2)		// 将 arr1 和 arr2 进行连接，并返回结果
alert(result)		// 显示 1,2,3,4,5
var result2 = arr1.concat(4)		// 讲 4 连接到 arr1 之后，并返回结果
alert(result2)		// 显示 1,2,3,4
```

## join
1. 在指定数组的每个元素之间插入值
2. 不操作数组本身，而是返回连接后的结果

```js
var arr = [1, 2, 3]
var result = arr.join('-')		// 在 arr 的每个元素之间添加 - 字符
alert(result)		// 显示 1-2-3
```

## sort
1. 方法本身存在缺陷，无法直接对数组进行正序或倒序的排序，需要自行传入方法进行辅助操作
2. 优化方式见 [Javascript 数组排序](http://asing1elife.com/javascript/array/2017/06/08/Javascript数组排序/)

```js
var arr = [10, 2, 4, 1, 7]
arr.sort()		// 并非对 arr 进行正序排序，而是讲数组中的每个元素视为字符串，讲其中的各位进行分别比较
 					// 例如将 10 分为 1 和 0 ，然后先将 1 和 2 4 7 比较，则认为 10 比 2 4 7 小
 					// 再将 1 和 1 比较，在相等的情况下则发现 10 比 1 还多一个 0 ，所以将 10 排在 1 之后
alert(arr)		// 显示 1,10,2,4,7
	 
// 用于辅助 sort() 进行排序的方法
// 若需要正序排序，则在 value1 < value2 的情况下返回 -1 ，在 value1 > value2 的情况下返回 1
// 若需要倒序排序，则在 value1 < value2 的情况下返回 1 ，在 value1 > value2 的情况下返回 -1
function compare(value1, value2) {
	if (value1 < value2) {
		return -1
	} else if (value1 > value2) {
		return 1
	} else {
		return 0
	}
}
	 
var arr = [10, 2, 4, 1, 7]
arr.sort(compare)		// 将写好的 compare 方法传入 sort() 中，则进行正序或倒序排序，具体情况根据 compare 中返回值决定
alert(arr)		// 显示 1,2,4,7,10
```

## reverse
1. 对数组中的内容进行反向输出

```js
var arr = [10, 2, 4, 1, 7]
arr.reverse()		// 直接将数组中的内容反向输出，而不是倒序
alert(arr)		// 显示 7,1,4,2,10
```

## indexOf(value)
1. 在指定数组中从第一个元素开始查询传入值的索引位置
2. 返回值为 -1 表示未找到对应值的索引

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var index = arr.indexOf(4)		// 在数组 arr 中查询数值 4 的索引位置，从第一个元素（索引为 0）开始计算，并返回索引值
alert(index)		// 显示 3
```

## indexOf(startIndex, value)
1. 从指定的索引开始，在指定数值中查找传入值的索引位置
2. startIndex 开始查询的索引
3. value 传入值

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var index = arr.indexOf(4, 4)		// 在数组 arr 中，从第 4 个索引开始，查询数值 4 的索引位置，并返回索引值
alert(index)		// 显示 5
```

## lastIndexOf(value)
1. 在指定数组中从最后一个元素开始查询传入值得索引位置

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var index = arr.lastIndexOf(2)		// 在数组 arr 中从最后一个元素开始查询数值 2 的索引位置，并返回索引值
alert(index)		// 显示 7
```

## lastIndexOf(startIndex, value)
1. 从指定的索引开始，在指定数组中从最后一个元素开始查询传入值的索引位置
2. startIndex 开始查询的索引
3. value 传入值

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var index = arr.lastIndexOf(2, 2)		// 在数组 arr 中从倒数第 2 个索引开始，查询数值 2 的索引位置，并返回索引值
alert(index)		// 显示 1
```

## every(function(item, index, array) { ... })
1. 对数组中的每个元素进行函数运算，并将总结果返回
2. 如果每个值得运算结果都返回 true ，则整体返回 true
3. 如果存在一个值运算结果返回 false ，则整体返回 false

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
// 方法体内存在一个回调函数
// item 为数值中当期元素
// index 为当前索引值
// array 为指定数组
var result = arr.every(function(item, index, array) {
	// 判断数组中的当前元素是否大于 2 ，结果返回 true 或 false
	return item > 2
})
alert(result)		// 显示 false
```

## filter(function(item, index, array) { ... })
1. 对数组中的每个元素进行函数运算，并将过滤后的结果返回

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var result = arr.filter(function(item, index, array) {
	// 判断数组中的当前元素是否大于 2 ，不满足条件则过滤掉
	return item > 2	
})
alert(result)		// 显示 3,4,5,4,3
```

## forEach(function(item, index, array) { ... })
1. 对数组中的每个元素进行遍历，并可在其中执行一个方法

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
// for 循环的升级版
arr.forEach(function(item, index, array) {
	alert(item)		// 循环显示 1 2 3 4 5 4 3 2 1	
})
```

## map(function(item, index, array) { ... })
1. 对数组中的每个元素进行函数运算，并返回新的数组

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var result = arr.map(function(item, index, array) {
	// 将数组中每个元素都乘以 2
	return item * 2
})
alert(result)		// 显示 2,4,6,8,10,8,6,4,2
```

## some(function(item, index, array) { ... })
1. 对数组中的每个元素进行函数运算，并将总结果返回
2. 如果存在一个得运算结果都返回 true ，则整体返回 true
3. 如果所有值运算结果返回 false ，则整体返回 false
4. 与 every() 逻辑正好相反

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var result = arr.some(function(item, index, array) {
	// 判断数组当前元素是否大于等于 5 ，返回 true 或 false
	return item >= 5
})
alert(result)		// 显示 true
```

## reduce(function(prev, cur, index, array) { ... })
1. 从左遍历数组的每个元素进行函数运算，并将总结果返回
2. prev 当前值的前一个值
3. cur 当前值
4. index 当前值的索引值
5. array 当前数组

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var result = arr.reduce(function(prev, cur, index, array) {
	// 当前值的前一个值 + 当前值 ，全部遍历结束后，再返回总和
	// 0 + 1 + 2 + 3 + 4 + 5 + 4 + 3 + 2 + 1
	return prev + cur	
})
alert(result)		// 显示 25
```

## reduceRight(function(prev, cur, index, array) { ... })
1. 从右遍历数组的每个元素进行函数运算，并将总结果返回
2. prev 当前值的前一个值
3. cur 当前值
4. index 当前值的索引值
5. array 当前数组

```js
var arr = [1, 2, 3, 4, 5, 4, 3, 2, 1]
var result = arr.reduceRight(function(prev, cur, index, array) {
	// 当前值的前一个值 + 当前值 ，全部遍历结束后，再返回总和
	// 0 + 1 + 2 + 3 + 4 + 5 + 4 + 3 + 2 + 1
	return prev + cur	
})
alert(result)		// 显示 25
```