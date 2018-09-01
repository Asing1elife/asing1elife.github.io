---
title: IntelliJ IDEA解决Tomcat部署项目NoClassDefFoundError报错
date: 2018-01-14
author: asing1elife
categories:
 - software
 - idea
tags:
 - software
 - idea
---
> 使用Tomcat部署Web项目时，项目所需要的jar包都会被复制到对应的lib目录下，但有时会出现jar包复制不完全的情况，则会出现java.lang.NoClassDefFoundError的报错  

## 将未添加到lib目录下的jar包进去添加
1. 按**Command+;**打开Project Structure窗口
2. 找到对应的项目后在Output Layout窗口中查看Available Elements
![](http://asing1elife.com/sources/images/EB3031F0-A8C7-4701-A52F-828E9FCDFFBE.png)
3. 打开对应项目，如果存在一些jar包列表则说明这些jar包没有被添加到lib目录下
4. 选中需要的jar包点击右键并put into _WEB-INF_lib即可
![](http://asing1elife.com/sources/images/6FCEA80D-0BD4-47DE-A26A-5BFD7C39F76D.png)