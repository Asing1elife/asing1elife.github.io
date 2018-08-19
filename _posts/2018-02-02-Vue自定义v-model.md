---
title: Vue自定义v-model
date: 2018-02-02
author: asing1elife
categories:
 - vue
tags:
 - vue
---
> Vue的v-model双向绑定非常实用，对于自定义组件也可以自定义实现v-model  

## 实现方案
1. 在组件的 `props` 中定义 `value` 值用于接收父级传入的内容

```javascript
props: {
  value: {
    type: Boolean,
    default: false
  }
}
```

2. 在组件的 `data()` 中定义将 `value` 另存为的实际值

```javascript
data () {
  return {
    show: false
  }
}
```

3. 在组件的 `watch` 中监听两个值的变化
	* 监听 `value` 变化的目的是将最新的值专递给组件中的实际值
	* 监听 `show` 变化的目的是当实际值发生变化时告知父容器传入值已发生变化

```javascript
watch: {
  value (val) {
    if (val === this.show) {
      return false
    }

    this.show = val
  },
  show (val) {
    this.$emit('input', val)
  }
}
```

4. 在父容器中使用 `v-model` 传入需要双向绑定的值即可
