---
title: Mint-UI Infinite Scroll 无限滚动运用
date: 2018-06-25
author: asing1elife
categories:
 - vue
 - mintui
tags:
 - vue
 - mintui
---
> Mint-UI 自带一个支持无限滚动的组件，以下是用法  

## 官网文档
[Infinite Scroll](https://mint-ui.github.io/docs/#/zh-cn2/infinite-scroll)

## 使用方法
1. `v-infinite-scroll` 指定加载内容的方法，在满足触发条件时，则会加载
	* 注意如果内容加载的分页方法就一个，此处指定后，则不需要再通过 `created()` 调用分页方法，否则一开始会加载两次
2. `infinite-scroll-distance` 触发方法的阈值，底部距离的像素值

```html
<div class="real-content" v-infinite-scroll="getMoocs" infinite-scroll-distance="10">
  <div class="hot-item" v-for="mooc in moocs" :key="mooc.hexId">
    ...
  </div>
</div>
```

## 注意点
1. 值得注意的是该组件提供了一个 `infinite-scroll-disabled` 属性，文档中的说明是**若为真，则无限滚动不会被触发**
	* 这很容易让人理解为当分页检测到没有下一页后，则不再触发无限滚动，但其实根本达不到这个效果
	* 翻阅 [官方 GitHub 上作者的解释](https://github.com/ElemeFE/vue-infinite-scroll/issues/25) 才知道这个属性其实为了防止分页内容在加载过程中重复加载，当内容加载完毕后，会被内部重置
	* 所以要实现到达尾页后不再重复发起分页请求，需要通过以下方法

```javascript
getMoocs() {
	// 检测是否存在下一页
  if (!this.page.hasNext) {
    return false
  }

  this.$fetch.mooc.page(this.page).then((res) => {
    this.page = new Page(res.data)

    this.moocs = _.concat(this.moocs, res.data.list)
  })
}

```