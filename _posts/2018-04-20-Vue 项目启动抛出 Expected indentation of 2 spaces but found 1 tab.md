---
title: Vue 项目启动抛出 Expected indentation of 2 spaces but found 1 tab
date: 2018-04-20
author: asing1elife
categories:
 - vue
 - eslint
tags:
 - vue
 - eslint
---
> 该问题是由于 ESLint 的验证规则不匹配  

## 解决方式
1. 在 **.eslintrc.js** 文件的 **rules** 中加入 `'no-tabs': 'off'` 即可不检测该问题

```javascript
// https://eslint.org/docs/user-guide/configuring

module.exports = {
	...
  'rules': {
		...
		'no-tabs': 'off'
  }
}
```