---
title: Linux rmdir
date: 2017-06-15
author: asing1elife
categories:
 - linux
tags:
 - linux
---
> rmdir-删除目录 只能删除非空目录  

## 删除目录
```shell
rmdir test
```

## 递归删除目录 
1. 若子目录删除后导致父目录为空，则会一并删除父目录

```shell
rmdir -p test/test1
```