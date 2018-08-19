---
title: Vue 集成 Animate.css
date: 2017-11-21
author: asing1elife
categories:
 - vue
tags:
 - vue
 - transition
 - animate
---
> 通过CSS实现了很多动画效果，可以直接调用  

## 官网
[Animate.css](https://daneden.github.io/animate.css/)

## 实现方式
1. 下载 **animate.css** 文件并在项目的 **main.js** 中引入

```javascript
import Vue from 'vue'

...

import 'assets/plugins/animate/animate.css'

...

/* eslint-disable no-new */
new Vue({
  el: '#app',
  render: h => h(App),
  store,
  router
})
```

## 结合 transition 组件一起使用
* `duration` 可控制动画的加载速度

```javascript
<transition leave-active-class="animated fadeOutRight" enter-active-class="animated fadeInRight" :duration="300">
  <mt-field class="search-input" placeholder="请输入搜索内容" v-show="search" v-model="searchContent"></mt-field>
</transition>
```