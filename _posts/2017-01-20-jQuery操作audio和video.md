---
title: jQuery操作audio和video
date: 2017-01-20
author: asing1elife
categories:
 - jquery
tags:
 - jquery
 - video
---
> 获取audio需要使用[0]，因为js操作获得的是audio对象，jQuery选择器获得的是jQuery对象，通过[0]取到的才是对应的节点对象  
> 所以不能直接使用jQuery对象去操作，需要通过[0]转换成js对应的节点对象  

## 实现方式
```
var audio = $("#audio")[0];
audio.pause();
audio.play();
```