---
title: DataGrip 隐藏 Collations
date: 2019-04-10
author: asing1elife
categories:
- software
- intellij
tags:
- software
- intellij
---
> DataGrip 在添加数据库连接后，默认会显示 schemas 和 collations  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 问题描述
1. schemas 是当前连接下的数据库列表，这是我们需要查看的东西
2. collations 是数据库的编码列表，这个对我们来说没有查看的必要
![](http://asing1elife.com/sources/images/90DDC87F-5E15-44BC-B961-3D29B6B6B201.png)

## 解决方式
1. 打开当前连接的配置页面，在 Options 选项卡中添加 `collation:-.*` ，保存后即可将 **collations** 文件夹隐藏
![](http://asing1elife.com/sources/images/DC1D4E1A-9236-4940-BE2E-685F4611F79C.png)