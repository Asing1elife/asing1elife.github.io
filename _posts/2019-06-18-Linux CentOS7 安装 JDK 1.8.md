---
title: Linux CentOS7 安装 JDK 1.8
date: 2019-06-18
author: asing1elife
categories:
- linux
- centos
- jdk
tags:
- linux
- centos
- jdk
---
> 介绍如何在 Linux CentOS7 中安装 JDK  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [centos7安装jdk环境 - 遁世不离俗 - 博客园](https://www.cnblogs.com/chy123/p/6750351.html)

## 安装步骤

### 下载 JDK
1. 前往 [Java SE Development Kit 8 - Downloads](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 下载对应版本的 Java for Linux 压缩包，如下图
	* 灰色高亮的就是正确下载项
![](http://asing1elife.com/sources/images/C24D0726-D181-4188-B9EA-47BD58807A5E.png)
2. 最好是使用下载工具，例如迅雷，压缩包下载好之后在通过 FTP 工具传输到服务器，例如 Transmit

### 解压 JDK 压缩包
1. 将下载好的 JDK 传输到服务器后，使用 `tar -zxvf jdk-8u211-linux-x64.tar.gz` 可以直接将 JDK 解压出来
2. 解压出来的内容就直接是对应的 JDK 完整目录，之后通过 `mv ./jdk1.8.0_211 /usr/local/java/jdk1.8.0_211` 将 JDK 移动到指定目录
	* 目录其实随意，但推荐放在一个统一目录
![](http://asing1elife.com/sources/images/3615790F-D47F-40F4-B9E7-988B61C0EB1A.png)

### 全局配置 JDK
1. 输入 `vim /etc/profile` ，在文件末尾添加以下内容

```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_211/
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```
2. 编辑好内容后，输入 `:wq` 保存并退出

### 重新加载配置
1. 输入 `source /etc/profile` 可以重新加载全局配置文件

### 测试 JDK 是否安装成功
1. 输入 `java -version` ，看到以下内容说明 JDK 安装成功
![](http://asing1elife.com/sources/images/997EE1E6-C697-45DC-82F9-6524A6CC42ED.png)