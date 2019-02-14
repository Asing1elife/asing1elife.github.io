---
title: Vue 项目启动抛出 Module build failed: Error: No ESLint configuration found.
date: 2018-04-19
author: asing1elife
categories:
 - vue
tags:
 - vue
 - eslint
---

> 项目启动时控制台抛出 `Module build failed: Error: No ESLint configuration found.` 的错误  

## 出现错误的原因

1. 因为项目根路径缺少 ESLint 的配置文件

## 解决问题的方式
1. 将 ESLint 相关配置文件添加至项目根路径
2. 文件名 **.eslintrc.js**
```javascript
// https://eslint.org/docs/user-guide/configuring

module.exports = {
  root: true,
  parser: 'babel-eslint',
  parserOptions: {
    sourceType: 'module'
  },
  env: {
    browser: true,
  },
  // https://github.com/standard/standard/blob/master/docs/RULES-en.md
  extends: 'standard',
  // required to lint *.vue files
  plugins: [
    'html'
  ],
  // add your custom rules here
  'rules': {
    // allow paren-less arrow functions
    'arrow-parens': 0,
    // allow async-await
    'generator-star-spacing': 0,
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0,
    // 忽略函数空格检测
    'space-before-function-paren': 0,
    // 中缀操作符周围要不要有空格
    'space-infix-ops': 0,
    'no-trailing-spaces': 0,
    'new-parens': 0
  }
}
```
3. 文件名 **.eslintignore**
```javascript
/build/
/config/
/dist/
/*.js
```

