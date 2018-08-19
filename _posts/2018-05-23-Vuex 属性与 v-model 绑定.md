---
title: Vuex 属性与 v-model 绑定
date: 2018-05-23
author: asing1elife
categories:
 - vue
 - vuex
tags:
 - vue
 - vuex
---
> 由于 Vuex 提供的是 mapGetters 和 mapActions 对属性行获取和操作，所以无法直接适配 v-model 的双向绑定形式  

## 官网
[Vuex 表单数据绑定 ](https://vuex.vuejs.org/zh/forms.html)

## 解决方式
1. 上述 vuex 的官方文档中提供了适配 v-model 的解决方式
2. 一种是重写 input 的双向绑定
3. 另外一种是提供单独的计算属性，并写明 get / set 方法，将 vuex 的 mapGetters / mapActions 分别赋值
4. 个人认为第二种方式更实用