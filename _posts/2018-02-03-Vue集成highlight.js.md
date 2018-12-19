---
title: Vue集成highlight.js
date: 2018-02-03
author: asing1elife
categories:
 - vue
 - highlight
 - javascript
tags:
 - vue
 - highlight
 - javascript
---
> highlight.js是一个比较实用的语法高亮插件，但其默认并不能在Vue中直接使用  

## 在项目的package.json文件中安装highlight.js
```javascript
{
	...
	"dependencies": {
		...
		"highlight.js": "^9.12.0"
	},
	...
}
```

## 编写集成文件，稍后会作为自定义插件引入Vue
```javascript
import Hljs from 'highlight.js'

import 'highlight.js/styles/googlecode.css'

let Highlight = {}

Highlight.install = (Vue, options) => {
  Vue.directive('highlight', (el) => {
    let blocks = el.querySelectorAll('pre code')
    
    blocks.forEach((block) => {
      Hljs.highlightBlock(block)
    })
  })
}

export default Highlight
```

## 在main.js中引入上述文件
```javascript
...
import Highlight from './assets/plugins/highlight/highlight.js'

Vue.use(Highlight)
...
```

## 在需要使用到语法高亮的标签上使用v-highlight并传入待显示内容
* 由于定义的监听规则是带 `pre code` 的内容，所以可能需要手动为待显示内容进行一次标签包裹

```html
<div v-highlight v-html="showFileContent"></div>
```

## 引入主题文件
1. 在插件的 `highlight.js/styles/` 目录下已经集成了多个主题
2. 选择合适的引入即可

```css
<style lang="stylus" rel="stylesheet/stylus">
  @import "~highlight.js/styles/atom-one-dark.css"
	...
</style>
```