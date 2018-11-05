---
title: 单体对象 Singleton Object 提供的顶层方法
date: 2018-11-05
author: asing1elife
categories:
 - javascript
tags:
 - javascript
 - object
---
> 单体对象也属于引用类型  

## 解释
1. Global 表示全局的对象，因为无法实例化，所以实际上是不存在的对象

## encodeURI 
1. 对传入值进行编码，会将一些不应存在于 URI 中的字符进行重新编码

```js
var uri = 'http://www.baidu.com cn'
var result = encodeURI(uri)
alert(result)		// 显示 http://www.baidu.com%20cn 表示空格被识别为 %20
```

## encodeURIComponent
1. 对传入值进行编码，会将一切对于浏览器而言不标准的字符进行重新编码

```js
var uri = 'http://www.baidu.com'
var result = encodeURIComponent(uri)
alert(result)		// 显示 http%3A%2F%2Fwww.baidu.com 表示 : 被识别为 %3A ，/ 被识别为 %2F
```

## decodeURI
1. 对传入值进行解码，与 encodeURI 规则相同

```js
var uri = 'http://www.baidu.com%20cn'
var result = decodeURI(uri)
alert(result)		// 显示 http://www.baidu.com cn 表示将 %20 解码为空格
```

## decodeURIComponent
1. 对传入值进行解码，与 **encodeURIComponent** 规则相同

```js
var uri = 'http%3A%2F%2Fwww.baidu.com'
var result = decodeURIComponent(uri)
alert(result)		// 显示 http://www.baidu.com 表示 %3A 被解码为 : ，%2F 被解码为 /
```

## eval
1. 无形的 JavaScript 解码器，可将普通字符串转换为 JS 代码

```js
var str = "var name = 'Jone'"
eval(str)		// 将 str 中的内容转换为 JS 代码
alert(str)		// 显示 Jone ，表示 name 变量已被声明
```

## eval('(' + strObj + ')')
1. 将对象字符串转换为对象，需要在传值的两侧加入一对括号，表示为一个单独方法域

```js
var strObj = "{name:'Jone', age:20}"
var obj = eval('(' + strObj + ')')
alert(obj.name + ', ' + obj.age)		// 显示 Jone, 20
```

## parseInt
1. 将字符串转换为 int 类型

```js
var num = parseInt('20')		// 将字符串 20 转换为 int 类型
alert(typeof(num) + ' : ' + num)		// 显示 number : 20
```

## parseFloat
1. 将字符串转换为 float 类型

```js
var num = parseFloat('20.5')		// 将字符串 20.5 转换为 float 类型
alert(typeof(num) + ' : ' + num)		// 显示 number : 20.5
```

## escape
1. 对传入的中文字符串进行转码

```js
var str = '王萌'
var result = escape(str)		// 对 王萌 进行转码，转成 Unicode 编码
alert(result)		// 显示 %u738B%u840c ，王 转换为 %u738B ，萌 转换为 %u840c
```

## unescape
1. 对传入的 Unicode 编码进行解码

```js
var str = '%u738B%u840c'
var result = unescape(str)		// 对字符串进行解码，转成中文
alert(result)		// 显示 王萌
```

## isNaN
1. 判断传入值是否为 number 类型，是 number 类型则返回 false ，不是 number 类型则返回 true

```js
var num1 = '10'
var num2 = 'abcd10e'
var num3 = '10abcde'
var result1 = isNaN(num1)		
var result2 = isNaN(num2)
var result3 = isNaN(num3)
alert(result1)		// 显示 false ，表明是 number 类型，而且 isNaN 还会对字符串进行自动转换
alert(result2)		// 显示 true ，表明不是 number 类型
alert(result3)		// 显示 true ，表明不是 number 类型，说明 isNaN 的自动转换机制并没有 parseInt 强大
```