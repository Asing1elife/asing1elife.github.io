---
title: Linux mkdir
date: 2017-06-15
author: asing1elife
categories:
 - linux
tags:
 - linux
---
> mkdir-创建目录  

## 创建目录
```shell
mkdir test
```

## 同时创建多个目录
```shell
mkdir -p test/test1
```

## 创建指定权限的目录
```shell
mkdir -m 777 test
```

## 创建目录并显示创建信息
```shell
mkdir -v test
```