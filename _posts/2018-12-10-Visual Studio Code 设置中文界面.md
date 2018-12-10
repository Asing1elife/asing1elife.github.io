---
title: Visual Studio Code 设置中文界面
date: 2018-12-10
author: asing1elife
categories:
 - vscode
 - software
tags:
 - vscode
 - software
---
> VS Code 虽然安装后默认是英文，但其实官方是支持中文的，毕竟 MS 的软件自带 12 国语言一直都是标配  

## 官方引导
[VS Code 界面语言设置](https://code.visualstudio.com/docs/getstarted/locales)

## 下载中文语言包
1. 在软件左侧菜单中打开 **Extensions** 插件商店
2. 搜索 **Language Pack** 就会出现一系列语言包，选择第一个 **中文(简体)** 即可
3. 安装语言包，结束后右下角会出现 **是否重启** 的提示，重启即可

![](http://asing1elife.com/sources/images/FFE5A66B-EEE7-4FB6-8373-104A847A4360.png)

## 安装语言包重启后系统界面语言没有修改
1. 按下 **CMD+Shift+P** ，输入 **Configure Display Language** ，并进入，会出现如下界面
2. 该文件默认只有 `"locale":"en"`
3. 将原内容注释并添加 `"locale":"zh-CN"` ，保存重启即可

![](http://asing1elife.com/sources/images/A22B6AE2-28DC-4A8D-A02E-428EC7E44500.png)