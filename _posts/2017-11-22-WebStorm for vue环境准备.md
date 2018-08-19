---
title: WebStorm for vue环境准备
date: 2017-11-22
author: asing1elife
categories:
 - vue
 - intellij
 - webstorm
tags:
 - vue
 - intellij
 - webstorm
---
> 在WebStorm中导入项目后，vue的语法默认不能被识别，所以需要做到如下配置  

## 插件
1. 输入 `Command+,` 进入**Preferences**界面，并找到**Plugins**分支
2. 点击页面底部的**Browse repositories**进入到插件商店
3. 在商店中搜索 `Vue` ，选中并安装
4. 重启WebStorm

## 语法
1. 输入 `Command+,` 进入**Preferences**界面，并找到**Languages Frameworks**分支
2. 打开分支选择第一项**JavaScript**
3. 将分支页面中的**JavaScript language version**修改为 `ECMAScript 6` 即可