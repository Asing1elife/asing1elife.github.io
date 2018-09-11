---
title: jQuery 解决 click 和 dblclick 冲突
date: 2018-09-11
author: asing1elife
categories:
 - jquery
 - javascript
tags:
 - jquery
 - javascript
---
> click 是单击事件， dblclick 是双击事件  
> 如果给一个 DOM 元素同时绑定两个事件，则会导致响应 dblclick 时会同时响应 click  

## 制定延迟策略
1. `set()` 用于为待执行方案设置一个延迟
2. `clear()` 用于清空设置的延迟

```js
var clickTimeout = {
  _timeout: null,
  set: function (fn) {
    var that = this
    that.clear()
    that._timeout = setTimeout(fn, 300)
  },
  clear: function () {
    var that = this
    if (that._timeout) {
      clearTimeout(that._timeout)
    }
  }
}
```

## 为单击事件添加延迟操作
1. 由于单击事件默认优先响应，所以需要为单击事件设置延迟策略，从而给双击事件足够的响应时间

```js
$('#btn').on('click', function () {
	clickTimeout.set(function () {
		console.log('this is click')
	})
})
```

## 在响应双击事件时清空延迟
1. 双击事件响应完毕后，清空延迟策略

```js
$('#btn').on('dblclick', function() {
	clickTimeout.clear()

	conosle.log('this is dblckick')
})
```