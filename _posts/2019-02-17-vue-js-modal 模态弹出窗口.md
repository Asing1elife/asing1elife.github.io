---
title: vue-js-modal 模态弹出窗口
date: 2019-02-17
author: asing1elife
categories:
- vue
- javascript
tags:
- vue
- javascript
---
> 基于Vue实现的Modal窗口，单独组件，方便使用，还很美观  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [vue-js-modal github](https://github.com/euvl/vue-js-modal)
2. [vue-js-modal example](http://vue-js-modal.yev.io)

## 使用方式

### 安装依赖
1. 使用终端在项目根目录运行以下脚本后，*package.json* 的 `dependencies` 中会出现 `vue-js-modal: ^x.x.xx` 的依赖

```sh
npm i -D vue-js-modal
```

### 全局注册插件
1. 在 *main.js* 中引入插件并注册到Vue全局实例中
2. 建议直接在配置中打开 `dialog` 以及 `dynamic` 的窗口模式，因为这两种模式默认是关闭的

```js
import VModal from 'vue-js-modal'

Vue.use(VModal, {
  dialog: true,
  dynamic: true
})
```

### 编写插件
1. 因为插件被全局注册，所以可以直接通过 `<modal>` 标签引用插件
2. 通常会指定窗口的名称，因为后续是通过名称调用窗口

```js
<template>
  <modal name="as-modal">
    这里显示窗口内容
  </modal>
</template>
```

### 调用插件
1. 使用 `import` 引入编写好的插件模块，同时注册到 `components` 中
2. 在 `<template>` 中实例化组件，组件如果有提供对外参数，此时也可以传入
3. 使用 `this.$modal.show('as-modal')` 打开窗口，这里的 *as-modal* 就是之前编写插件时为 `<modal>` 指定的名称

```js
<template>
  <div id="organizationPanel" class="body">
    <div class="organization-item new" @click="openModal"></div>
    <as-modal/>
  </div>
</template>

<script>
  import asModal from 'components/as-modal'

  export default {
    name: 'organization',
    methods: {
      openModal () {
        this.$modal.show('as-modal')
      }
    },
    components: {
      asModal
    }
  }
</script>
```

## 部分API的使用和注意点

### 属性
1. *name* 指定窗口名称
	* 显示窗口时将名称传入 `this.$modal.show(name)` 函数即可
2. *classes* 指定窗口className
	* 注意不是 *class* 
	* 查看源码得知本属性支持传入 *String* 或 *Array* ，默认值是 `v--modal`
	* 所以如果传入自定义className ，属于默认值的 `v--modal` 就不会被指定，也就导致默认的样式无法生效
	* 建议在传入自定义className时，同时指定默认值，例如 `as-modal v--modal` ，这样自定义和默认值都会生效
3. *transition* 窗口弹出的过渡效果
	* 	没有默认值，但原生提供了 `nice-modal-fade` 效果，可以直接显示指定
	* 想要其他效果需要自定义编写，命名和使用规则参照Vue过渡效果
4. *overlayTransition* 遮罩层过渡效果
	* 	默认值 `overlay-fade`
	* 逍遥其他效果需要自定义编写，命名和使用规则参照Vue过渡效果