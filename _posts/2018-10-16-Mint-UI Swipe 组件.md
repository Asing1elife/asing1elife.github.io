---
title: Mint-UI Swipe 组件
date: 2018-10-16
author: asing1elife
categories:
 - vue
 - mintui
tags:
 - vue
 - mintui
--- 
> Mint-UI 提供一个 Swipe 组件用于实现轮播，但官方文档实在过于简陋，此处通过翻阅源码查找到以下实用方法  

## 官方文档
1. [Swipe 文档](https://mint-ui.github.io/docs/#/zh-cn2/swipe)
2. [Swipe 源码](https://github.com/ElemeFE/mint-ui/blob/master/packages/swipe/src/swipe.vue)

## 禁止自动滚动
1. 控制自动滚动的参数是 `auto` ，默认值 3000 毫秒
2. 在 API 文档中没有说明如何禁止自动滚动，但是在示例代码中有说明将 `:auto="0"` 即可禁止自动滚动

```html
<mt-swipe :auto="0">
  <mt-swipe-item>...</mt-swipe-item>
</mt-swipe>
```

## 手动切换上 / 下一个
1. API 文档中没有任何关于操作组件的方法，但在源码中可以翻阅到如下代码

```js
next() {
	this.doAnimate('next')
},
prev() {
	this.doAnimate('prev')
},
```
2. 所以可以通过为组件添加 `ref` ，并调用以上方法来实现上下子项的切换

```html
<mt-swipe ref="testSwipe"></mt-swipe>
```
```js
// 手动切换到下一个
this.$refs.testSwipe.next()
// 手动切换到上一个
this.$refs.testSwipe.prev()
```