---
title: vue-aliplayer 阿里云播放器适配 vue [新]
date: 2019-04-04
author: asing1elife
categories:
- vue
- javascript
- aliplayer
tags:
- vue
- javascript
- aliplayer
---
> 阿里云的点播/直播播放器本身没有对 vue 进行支持，以下会对 aliplayer 做简单封装，让其支持 vue  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 之前写过的内容
1. [vue-aliplayer 阿里云播放器适配 vue](http://asing1elife.com/vue/2018/05/02/vue-aliplayer-阿里云播放器适配-vue/)
2. 上文是之前写的一个让 aliplayer 适配 vue 的笔记，是基于 github 上一个现成的库做的二次封装
3. 但实际使用后发现其本身有一些缺陷，而且大部分封装功能自己也用不到
4. 所以这次直接引入 aliplayer 的源码直接进行封装，实现按需开发

## 相关网站
1. [阿里云Aliplayer播放器](https://player.alicdn.com/aliplayer/tutorial/tutorial.html)

## 实现效果
1. 底部的输入框直接使用的 Element-UI 的组件，输入 vid 后按右侧按钮可以实现快速切换视频源
![](http://asing1elife.com/sources/images/285E05F4-0164-482D-A445-D9B027CEF185.png)

## 具体实现
1. `inIcon` 只是一个自行封装的图标组件，内部使用的是 font-awesome 的图标库，不影响播放器组件本身实现
2. 输入并切换视频 id 的输入框和按钮直接使用的 Element-UI 组件，不影响播放器组件本身实现

```js
<template>
  <div class="in-player-panel">
    <div class="in-player"
         :id="playerId"></div>
    <el-input placeholder="请输入视频vid" v-model="videoId">
      <el-button slot="append"
                 @click="changePlayAuth">
        <in-icon name="play-circle"></in-icon>
      </el-button>
    </el-input>
  </div>
</template>

<script type="text/ecmascript-6">
  import inIcon from 'components/in-icon/in-icon'

  // aliplayer h5 格式播放器的 js 地址
  const jsPath = 'http://g.alicdn.com/de/prismplayer/2.8.1/aliplayer-h5-min.js'
  // aliplayer h5 格式播放器的 css 地址
  const cssPath = 'http://g.alicdn.com/de/prismplayer/2.8.1/skins/default/aliplayer-min.css'

  // 视频的长宽比例，采用 width / height 得到，基本按照 16:9 获取
  const ratio = 1.78

  export default {
    name: 'in-player',
    props: {
      // 父容器传入的视频 vid
      vid: String,
      // 父容器传入的播放器宽度
      width: Number,
      // 父容器传入的播放器高度
      height: Number
    },
    data () {
      return {
        // 内部真是的 vid
        videoId: this.vid,
        // 默认宽度
        defaultWidth: 718,
        // 默认高度
        defaultHeight: 400,
        // 每个播放器都具有单独的 id
        playerId: 'inPlayer' + Math.random() * 100000000000000000,
        // 通过 vid 从后端获取到的鉴权地址
        playAuth: null,
        // 播放器实例
        inPlayer: null
      }
    },
    computed: {
      // 计算后得到的真实宽度
      realWidth () {
        // 如果从外部传入了高度，则按照外部高度乘以宽高比，获取真实宽度，否则按照默认高度计算
        return `${(this.height || this.defaultHeight) * ratio}px`
      },
      // 计算后得到真实高度
      realHeight () {
        // 如果从外部传入了宽度，则按照外部宽度乘以宽高比，获取真实高度，否则按照默认宽度计算
        return `${(this.width || this.defaultWidth) / ratio}px`
      }
    },
    mounted () {
      // 在 mounted() 中初始化播放器，是为了确保基本元素都已经加载
      this._initialize()
    },
    methods: {
      // 初始化
      _initialize () {
        // 判断播放器全局实例是否已存在，只有当播放器依赖的 JS 加载完成后，播放器的全局实例才存在
        if (!window.Aliplayer) {
          // 尝试获取播放器的 JS 标签
          let inPlayerScriptTag = document.getElementById('inPlayerScriptTag')

          // 判断JS是否已存在
          if (!inPlayerScriptTag) {
            // 不存在则创建 JS 标签
            inPlayerScriptTag = document.createElement('script')
            inPlayerScriptTag.type = 'text/javascript'
            // 指定 JS 的地址
            inPlayerScriptTag.src = jsPath
            // 指定 JS 的 ID
            inPlayerScriptTag.id = 'inPlayerScriptTag'

            // JS 不存在说明 CSS 也不存在，则创建 CSS 标签
            let inPlayerLinkTag = document.createElement('link')
            inPlayerLinkTag.type = 'text/css'
            inPlayerLinkTag.rel = 'stylesheet'
            // 指定 CSS 的地址
            inPlayerLinkTag.href = cssPath

            // 获取页面的 <head> 标签
            let head = document.getElementsByTagName('head')[0]
            // 将 JS 和 CSS 插入到 DOM 中
            head.appendChild(inPlayerScriptTag)
            head.appendChild(inPlayerLinkTag)
          }

          // JS 插入并加载完成
          if (inPlayerScriptTag.loaded) {
            // 初始化播放器
            this._initPlayer()
          } else {
            // JS 没有加载完成，则监听 JS 的加载事件
            inPlayerScriptTag.addEventListener('load', () => {
              // JS 确认加载完成后，初始化播放器
              this._initPlayer()
            })
          }
        } else {
          // 全局实例存在则直接初始化播放器
          this._initPlayer()
        }
      },
      // 初始化播放器
      _initPlayer () {
        // 确保 DOM 元素都已经渲染完毕
        this.$nextTick(() => {
          // 判断播放器实例是否存在
          if (!this.inPlayer) {
            // 获取鉴权地址，从后端获取
            this.$fetch.video.auth({
              vid: this.videoId
            }).then((res) => {
              // 接收从后端获取的鉴权地址，后端实现不赘述
              this.playAuth = res.data

              // 初始化播放器
              this.inPlayer = window.Aliplayer({
                // 播放器 ID
                id: this.playerId,
                // 视频的 VID
                vid: this.videoId,
                // 鉴权地址
                playauth: this.playAuth,
                // 宽度
                width: this.realWidth,
                // 高度
                height: this.realHeight,
                // 使用 H5 格式
                useH5Prism: true,
                // 不是直播 
                isLive: false,
                // 不自动播放
                autoplay: false
              })
            })
          } else {
            // 存在播放器实例则先销毁组件
            this.inPlayer.dispose()
            // 将播放器实例重置
            this.inPlayer = null
            // 清空播放器 DOM 内容
            document.getElementById(this.playerId).innerHTML = ''

            // 再次初始化
            this._initPlayer()
          }
        })
      },
      // 更新播放地址
      changePlayAuth () {
        // 重新初始化
        this._initPlayer()
      }
    },
    components: {
      inIcon
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "~assets/stylus/variable.styl"

  .in-player-panel
    display flex
    flex-direction column
    .in-player
      flex-grow 1
      border 1px solid $color-border
      border-bottom none
      border-top-left-radius 4px
      border-top-right-radius 4px
      overflow hidden
    .el-input
      flex 0 0 40px
      .el-input__inner
        border-top-left-radius 0
      .el-input-group__append
        border-top-right-radius 0
        .in-icon
          font-size $text-size-h
</style>
```