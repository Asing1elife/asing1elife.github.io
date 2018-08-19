---
title: Vue 项目启动抛出 Error/ No PostCSS Config found in
date: 2018-04-20
author: asing1elife
categories:
 - vue
tags:
 - vue
---
> 项目启动时抛出 Error: No PostCSS Config found in … 的错误表示某个 css 文件不能被引入  

## 解决办法
1. 在项目根目录创建 **postcss.config.js** 文件，并配置以下内容

```javascript
module.exports = {  
  plugins: {  
    'autoprefixer': {browsers: 'last 5 version'}  
  }  
} 
```