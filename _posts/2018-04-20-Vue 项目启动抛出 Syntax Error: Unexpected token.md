---
title: Vue 项目启动抛出 Syntax Error/ Unexpected token
date: 2018-04-20
author: asing1elife
categories:
 - vue
tags:
 - vue
---
> 项目启动时抛出了标识符无法识别的错误  

## 错误原因
1. ES6 新增了不少标识符，但浏览器大多无法直接识别，需要借助 **babel** 对 ES6 代码进行转义
2. 项目启动时抛出如下错误，表示 `...` 运算符没能被识别，该运算符属于 ES6 的解构运算符
3. 出现该问题的原因基本上可以定位是项目没有配置 **babel**
	* 即时 **package.json** 文件中已经引入 **babel** ，但仍然需要在项目根目录创建一个 **.balbelrc** 文件进行配置
![](http://asing1elife.com/sources/images/B426AA54-B759-437C-B914-BF2AEDC7E05E.png)

## 解决方式
1. 在项目根目录创建 **.babelrc** 文件即可实现对 **babel** 的基本配置

```.babelrc
{
  "presets": [
    ["env", {
      "modules": false
    }],
    "stage-2"
  ],
  "plugins": ["transform-runtime", "transform-vue-jsx"]
}
```

