---
title: preventDefault 和 stopPropagation 的区别
date: 2018-09-04
author: asing1elife
categories:
 - javascript
tags:
 - javascript
---
> 在使用 JS 阻止默认动作时，通常会使用 preventDefault 或 stopPropagation ，但两者存在一定区别  

## preventDefault
1. 如下方法中，`.training-name-input` 存在与一个表单中，该元素监听了键盘的回车事件
2. 如果不使用 `e.preventDefault()` 则会在触发该监听的同时，触发表单中 `type="submit"` 按钮的提交事件
3. 所以 `preventDefault` 的关键作用在于通知浏览器不执行与事件关联的默认动作

```js
$diplomaContentPanel.find('.training-name-input').on('keydown', function (e) {
  e.preventDefault()

  if (e.keyCode === 13) {
		// 只响应回车事件
  }
})

```

## stopPropagation
1. 如下方法中，`carousel-indicators li` 是一个插件的节点，本身就被插件赋予了点击事件
2. 但是使用时希望屏蔽该插件赋予的点击事件，这个时候就需要使用 `e.stopPropagation()`
3. 作用在于，阻止 DOM 元素继续向下冒泡，在执行了该点击方法后，就停止了，不再响应被赋予的其他点击事件

```js
$noviceGuidePanel.find('.carousel-indicators li').on('click', function (e) {
	e.stopPropagation()
})
```