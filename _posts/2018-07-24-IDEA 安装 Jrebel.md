---
title: IDEA 安装 Jrebel
date: 2018-07-24
author: asing1elife
categories:
 - software
 - idea
tags:
 - software
 - idea
 - jrebel
---
> 虽然 IDEA 本身支持热部署，但功能较弱，只支持方法内部代码更新，如果新增方法则需要重启项目，使用 Jrebel 可以实现完全的项目热部署  

## Jrebel 安装
1. Jrebel 提供了 IDEA 的插件支持，进入插件列表搜索下载即可
2. 但 Jrebel 不是免费的，需要激活
![](http://asing1elife.com/sources/images/48D8F421-3809-4CEE-8E56-1F93DFF55236.png)

## 为什么要激活 Jrebel 
1. 之前的 Jrebel 有个人版，支持 Facebook 账号的免费激活使用，但目前 Jrebel 更新到 2018.1.4 版本后已经下线了个人版，只提供企业版
2. 企业版 Jrebel 的价格实在有点贵，而且也是按年付费
	
## Jrebel 实现本地激活
### 下载激活文件
	1. 前往 [Jrebel License Server](https://github.com/ilanyu/ReverseProxy/releases/tag/v1.4) 下载 ReverseProxy_darwin_amd64 该文件是 mac 可使用的

### 为文件添加权限并运行
```shell
chmod 777 ./ReverseProxy_darwin_amd64
$bash ./ReverseProxy_darwin_amd64
```

### 运行成功后显示如下内容
![](http://asing1elife.com/sources/images/0B507B93-67DE-4132-A95D-E736C79330D8.png)

### 前往 Jrebel 配置界面按如下输入
1. 地址后面需要跟一串 [GUID](https://www.guidgen.com) 验证
2. 邮箱可随意输入
![](http://asing1elife.com/sources/images/35FF31AD-AE06-4452-9743-C5290184AEB8.png)
	
### 输入成功显示激活后要立即将 Jrebel 设置为离线模式，可确保 180 有效性
1. 180 天后重复上述操作即可
2. 如果不设置为离线模式，终端运行的程序就不能退出，一旦退出激活就会失效 ![](http://asing1elife.com/sources/images/1FD212E3-4314-4287-967F-15A15B119996.png)