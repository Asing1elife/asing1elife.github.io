---
title: vuex 在非组件中调用 actions 方法
date: 2018-08-14
author: asing1elife
categories:
 - vue
 - vuex
tags:
 - vue
 - vuex
---
> 一般情况下调用 actions.js 中的方法都是在组件中，如果想在非组件中调用，则需要如下方式  

## 在组件中
```javascript
import {mapActions} from 'vuex'

export default {
	methods: {
		...mapActions([
			'setUserInfo'
		])
	},
	init() {
		let user = {}
		
		this.setUserInfo(user)
	}
}
```

## 在非组件中
```javascript
import store from 'store'

function init() {
	let user = {}

	store.dispatch('setUserInfo', user)
}
```