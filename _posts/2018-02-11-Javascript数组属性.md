---
title: Javascript数组属性
date: 2018-02-11
author: asing1elife
categories:
 - javascript
 - array
tags:
 - javascript
 - array
---
> Javascript数组可用来存储多个数组，但他也有些不常见的内置功能  

## 数组的真面目
1. 数组的索引其实也是数组的属性，所以如下操作是等同的

```javascript
let array = ['Tom', 'Jerry']

console.log(array[0]) => Tom
console.log(array.0) => Tom 
```

2. 数组的内置属性

```javascript
let array = ['Tom', 'Jerry']

console.log(array.length) => 2
console.log(array['length']) => 2
```

3. 数组其实可以添加自定义属性，因为数组其实也是一个object对象

```javascript
let array = ['Tom', 'Jerry']

array.itemName = 'wow'

console.log(array.itemName) => wow
```

## 循环数组的元素
1. 自从ES6发布之后，可以不再使用传统的 `for` 循环对数组进行遍历，而可以使用 `for ... of` 循环直接操作数组元素

```javascript
for (let item of array) {
	console.log(item)
}
```

## 数组元素的数量不等于数组长度
1. 通常情况下数组元素的数量就是数组的长度，但这种关系非常脆弱，参见如下代码

```javascript
let array = []

array.lentgh = 3

console.log(array.length) => 3

array[5] = 'Lily'

console.log(array.length) => 5
```

## 删除数组元素
1. 删除数组元素的方法不是 `remove()` ，而应该使用 `splice()`
2. `splice(index, number, item...)`  提供三个属性
	* **index** 表示待删除元素的索引
	* **number** 表示从开始删除元素的索引开始要删除几个元素，1 表示只删除该元素，0 表示不删除
	* **item…** 可选，可传入多个新元素，将从被删除元素的索引位置插入
3. `splice()` 的特点在于会直接作用原数组，返回值中返回的是被删除元素

```js
var arr = ['java', 'html', 'css', 'js', 'vue']

// java,html,css,js,vue
console.log(arr)

// 删除索引1的元素
var result = arr.splice(1, 1)

// java,css,js,vue
console.log(arr)
// html
console.log(result)
```