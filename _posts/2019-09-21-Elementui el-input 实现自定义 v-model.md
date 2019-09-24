---
title: Elementui el-input 实现自定义 v-model
date: 2019-09-21
author: asing1elife
categories:
- vue
- javascript
tags:
- vue
- javascript
---
> Vue 本身支持自定义组件实现 v-model ，但 el-input 作为 Elementui 的自定义组件也已经实现 v-model ，那么如果在自定义组件中使用 el-input ，但自定义组件也想实现 v-model ，应该怎么做？  

## 错误的方式
1. Vue 中让自定义组件实现 v-model 可参考 [Vue自定义v-model](http://asing1elife.com/vue/2018/02/02/Vue自定义v-model/)
2. 但如果按照这种方式想要让以下代码实现 v-model ，是不可取的
	* 可以用，但每次在页面第一次渲染的时候都没有值，非要手动输入一次，才会触发变化
	* 这是因为下面这个 in-player 组件中的 el-input 已经实现了自定义的 v-model ，当 in-player 再次通过同样的方式实现时，就出现了两次 **watch** 操作

```js
<template>
  <div class="in-player-panel">
    <el-input placeholder="请输入视频vid" v-model="videoId">
      <el-button slot="append" @click="changePlayAuth">播放</el-button>
    </el-input>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    name: 'in-player',
    props: {
      value: {
        type: String,
        value: ' '
      }
    },
    data () {
      return {
        videoId: ''
      }
    },
    watch: {
      'value' (val) {
        if (val == this.videoId) { return false }
        this.videoId = val
      },
      'videoId' (val) { this.$emit('input', val) }
    }
  }
</script>
```

## 正确的方式
1. 如果在自定义组件中，既想使用 el-input 的样式和规则，又想让组件本身实现自定义 v-model
2. 那么就应该像如下代码一样，不直接使用 el-input 的 v-model 实现，转而使用其 `@input` 函数

```js
<template>
  <div class="in-player-panel">
    <el-input placeholder="请输入视频vid" :value="value" @input="update">
      <el-button slot="append" @click="changePlayAuth">播放</el-button>
    </el-input>
  </div>
</template>

<script type="text/ecmascript-6">
  export default {
    name: 'in-player',
    props: {
      value: {
        type: String,
        value: ' '
      }
    },
    methods: {
      update (val) {
        this.$emit('input', val)
      }
    }
  }
</script>
```