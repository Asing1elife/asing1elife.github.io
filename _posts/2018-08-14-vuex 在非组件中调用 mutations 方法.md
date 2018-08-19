---
title: vuex 在非组件中调用 mutations 方法
date: 2018-08-14
author: asing1elife
categories:
 - vue
 - vuex
tags:
 - vue
 - vuex
---
> 一般情况下调用 mutations.js 中的方法都是在组件中，如果想在非组件中调用，则需要使用如下方式  

## 在组件中调用
```javascript
import {mapMutations} from 'vuex'
import {SET_IS_LOGIN} from 'store/mutation-types'

export default {
	methods: {
		...mapMutations({
			set_is_login: SET_IS_LOGIN
		}),
		init() {
			this.set_is_login(true)
		}
	}
}
```

## 在非组件中调用
```javascript
import store from 'store'
import {SET_IS_LOGIN} from 'store/mutation-types'

function init() {
	store.commit(SET_IS_LOGIN, true)
}
```