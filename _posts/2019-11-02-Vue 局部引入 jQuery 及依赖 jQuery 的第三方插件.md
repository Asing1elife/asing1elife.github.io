---
title: Vue 局部引入 jQuery 及依赖 jQuery 的第三方插件
date: 2019-11-02
author: asing1elife
categories:
- vue
- jquery
tags:
- vue
- jquery
---
> 虽然在 Vue 中不推荐使用 jQuery ，但有时候要使用一个插件，这个插件又依赖于 jQuery ，则没有更好的办法  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. 如果实在是因为无法避免的原因，需要在 Vue 项目中使用到 jQuery
2. 则建议使用局部引入的方式，就是哪个文件需要用到 jQuery ，就在哪个文件中引入
3. 在网上搜 “Vue 引入 jQuery” ，能找到的基本上都是全局引入的方式
	* 会涉及到需要修改 webpack 的配置文件
4. 或者使用 **expose-loader** 的插件来实现 jQuery 的局部引入
5. 其实没必要，就普通的引入方式即可

## 解决方式
### 安装 jQuery
1. 在项目的 **package.json** 中安装 jQuery ，这个是必须的
	* 之后运行 `npm i` 即可实现安装

```json
“dependencies”: {
  ...
  “jquery”: “^1.12.4”
}
```

### 引入第三方插件
1. 前往需要用到插件的 `.vue` 文件中，在 `<script>` 顶部引入以下代码
2. 由于 jQuery 是使用 webpack 安装的，所以通过 import 引入，并指定别名
3. 插件则是本地引入的，所以使用 require 的方式

```js
import $ from ‘jquery’

window.jQuery = $
require(‘assets/plugins/pictureTag/picture.tag’)
```
4. 添加 `window.jQuery = $` 则是因为插件内部会需要使用名为 **jQuery** 的引用

```js
(function (window, $) {
...
})(jQuery(window), jQuery)
```

### 使用第三方插件
1. 由于 jQuery 相关插件基本上涉及到操作 DOM 元素
2. 所以需要在 `$nextTick` 中使用
3. 之后按照传统的 jQuery 操作方式编写插件调用代码即可

```js
this.$nextTick(() => {
  ...
})
```