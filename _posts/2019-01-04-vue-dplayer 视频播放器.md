---
title: vue-dplayer 视频播放器
date: 2019-01-04
author: asing1elife
categories:
 - vue
tags:
 - vue
---

> vue-dplayer 是对 dplayer 做的 vue 支持  

## 官网
1. [vue-dplayer](https://github.com/MoePlayer/vue-dplayer)
2. [dplayer-doc](http://dplayer.js.org/#/home?id=options)

## 示例
1. 如果默认 `options` 中没有视频链接，之后设置视频链接时，直接通过 `this.options.video.url = videoPath` 是无效的
2. 需要先获取到播放器的实例 `this.$refs.player.dp` ，然后通过 `switchVideo()` 对 `url` 进行修改



```js
<template>
  <d-player ref="player" :options="options"></d-player>
</template>

<script type="text/ecmascript-6">
  import dPlayer from 'vue-dplayer'
  import 'vue-dplayer/dist/vue-dplayer.css'

  export default {
    name: 'in-video',
    props: {
      source: {
        type: String,
        default: ''
      }
    },
    data () {
      return {
        player: null,
        options: {
          video: {
            url: ''
          },
          contextmenu: [
            {}
          ]
        }
      }
    },
    mounted() {
      this.player = this.$refs.player.dp
    },
	  created() {
      this._setVideoUrl(this.source)
    },
    methods: {
      // 设置视频播放路径
      _setVideoUrl (url) {
        this.player.switchVideo({
          url: url
        })
      }
    },
    components: {
      dPlayer
    }
  }
</script>
```