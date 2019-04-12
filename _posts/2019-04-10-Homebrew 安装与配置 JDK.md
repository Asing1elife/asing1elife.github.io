---
title: 
date: 
author: asing1elife
categories:
tags:
---
> 通过 Homebrew 快捷安装 指定版本 JDK  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 常规操作
1. 通过 HomeBrew 安装 JDK 一般直接在终端运行以下内容即可
2. 但这里其实有一个非常大的坑，**直接安装默认指向的是最新版本，截止目前，通过以下方式安装的 JDK 版本是 12**
3. 老实说我现在项目最新用的也就 8 ，手中还维护着几个使用 7 的项目，所以这个方式显然不可行

```sh
brew cask install java
```

## 非常规操作安装指定版本 JDK

### 查看支持下载的 JDK 列表
1. 在终端执行 `brew search java` 可以查看当前可以支持下载的 java 相关内容列表
2. 显示出来的内容如下图，只有一个 **java** 是指代的 JDK ，而这个版本的 JDK 就是前文中所述的最新版本
![](http://asing1elife.com/sources/images/3F8E602B-FE73-4943-9CF6-FD8536FED5A9.png)

### 获取 Homebrew 历史版本库
1. 其实 Homebrew 是支持下载对应软件的历史版本的，只是它的历史版本库默认没有被链接到本地
2. 在终端执行 `brew tap homebrew/cask-versions` 可以获取 Homebrew 的历史版本库
	* 这个方法是在 [How do I install Java on Mac OSX allowing version switching? - Stack Overflow](https://stackoverflow.com/questions/52524112/how-do-i-install-java-on-mac-osx-allowing-version-switching) 里找到的
3. 之后再执行 `brew search java` ，即可看到如下内容
4. 从下图中就可以看到出现了 **java** 、**java6** 、**java8** 、**java11** 
	* java8 后面打勾是表示我已经安装了这个版本的 JDK
5. 现在再执行 `brew cask install java8` ，即可完成 JDK 版本为 8.x 的安装
![](http://asing1elife.com/sources/images/10ACBE05-078D-485B-9162-966E4E2180F1.png)

### 为什么找不到 JDK 7.x 的安装包
1. 如果对 JDK 7.x 有需求的应该会发现上图中没有提供对应的 **java7** 安装包
2. 前文中说过我手上还有几个 JDK 7.x 的项目需要维护，所以这让我非常郁闷，其实去 Oracle 官方下载对应版本的 JDK 应该是最快捷的方式，但这个时候强迫症就犯病了，无法忍受两个 JDK 安装的方式不同
3. 接下来在 [Remove java7 by commitay · Pull Request #3914 · Homebrew/homebrew-cask-versions · GitHub](https://github.com/Homebrew/homebrew-cask-versions/pull/3914) 里找到了解决方案
	* 这里面貌似说的是 java7 的安装包被移除了，但是文章最末尾提供了一个替代解决方案
4. 按文中所述，在终端执行 `brew cask install caskroom/versions/zulu7` 其实下载安装的就是 JDK 7.x 
5. 虽然安装之后显示的名字很奇怪，但至少是一个可行的方案
	* 有的人可能要问你不是强迫症吗，这名字不同你就受得了？
	* 科科，貌似真受得了
6. 总之安装之后执行 `/usr/libexec/java_home -V` ，可看到下图，说明是安装成功的
![](http://asing1elife.com/sources/images/F63BEFB3-E044-4D5A-9C83-204C2AC40E2B.png)