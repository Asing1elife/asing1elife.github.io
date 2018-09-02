---
title: Linux alias
date: 2017-06-15
author: asing1elife
categories:
 - linux
tags:
 - linux
---
> alias用于给命令添加一个简洁的别名  

## 添加永久性别名
1. 输入 `vi ~/.bashrc`
2. 在文件中添加需要的命令别名，例如 `alias ll='ls -l'` 并保存
3. 输入 `source ~/.bashrc` 执行文件
4. 将 `source ~/.bashrc` 写入 `vi ~/.bash_profile` 文件中并保存，可确保重启后命令依旧生效