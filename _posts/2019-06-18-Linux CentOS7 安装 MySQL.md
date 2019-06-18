---
title: Linux CentOS7 安装 MySQL
date: 2019-06-18
author: asing1elife
categories:
- linux
- centos
- mysql
tags:
- linux
- centos
- mysql
---
> 介绍如何在 Linux CentOS7 中在线安装 MySQL 8.x  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/)
2. [MySQL :: A Quick Guide to Using the MySQL Yum Repository](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)
3. [CentOS7下安装MySQL5.7安装与配置（YUM） - 先定一个小目标 - 博客园](https://www.cnblogs.com/lgqboke/p/6873734.html)

## 安装步骤

### 下载 8.x 的安装包
1. 下载地址的前缀是 `wget http://dev.mysql.com/get/`
2. 后面拼接具体版本的安装包名称即可，如下图，是在 [MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/) 中找到的安装包信息
![](http://asing1elife.com/sources/images/5716FCBE-B665-49E8-92F9-DC838154CDEE.png)
3. 将名称拼接后完整下载命令是 `wget http://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm`

### 解压安装包
1. 输入 `sudo rpm -Uvh mysql80-community-release-el7-3.noarch.rpm`  或者 `yum install mysql80-community-release-el7-3.noarch.rpm` 对安装包进行解压
![](http://asing1elife.com/sources/images/CDAD6E94-8F57-41F3-9109-912CC72DFDA9.png)

### 确认安装版本
1. 输入 `yum repolist all | grep mysql` 可以查看当前 安装包中分别 **启用/禁用** 了哪些版本的内容

### 安装对应版本的 MySQL
1. 输入 `sudo yum install mysql-community-server` 即开始安装
2. 过程中需要下载很多依赖包，耐心等待即可，如下图，真的很慢
![](http://asing1elife.com/sources/images/1222945D-26D5-405C-BC56-BE3C170234BE.png)

### 启动 MySQL 服务
1. 参考 [MySQL 命令大全](http://asing1elife.com/database/mysql/2017/06/19/MySQL-命令大全/)

### 配置 MySQL 自启动
1. 输入即可，没有任何提示

```sh
systemctl enable mysqld
systemctl daemon-reload
```

### 修改 root 用户密码
1. 输入 `grep 'temporary password' /var/log/mysqld.log` 可以看到 MySQL 在安装时分配的默认密码，如下图
	* 绿色高亮的部分就是临时密码
![](http://asing1elife.com/sources/images/A18E571D-B5FF-47E0-BB65-FA04FD039B7F.png)
2. 输入 `mysql -uroot -p` 之后，使用上述密码登录数据库
3. 在数据库终端输入 `alter user 'root'@'localhost' identified by 'newPassword';` 即可修改密码
	* 在 [CentOS7下安装MySQL5.7安装与配置（YUM） - 先定一个小目标 - 博客园](https://www.cnblogs.com/lgqboke/p/6873734.html) 有关于密码安全策略的提示，如下图
![](http://asing1elife.com/sources/images/E89CCE88-B09B-460E-A78D-D447825316B2.png)

### 允许用户远程登录
1. 输入 `create user 'asing1elife'@'%' identified by 'newPassword';` 创建一个远程登录用户
2. 输入 `grant all on *.* to 'asing1elife'@'%';` 将所有权限给新创建的用户

### 远程登录可能碰到的问题
1. 使用新用户进行远程登录时可能会抛出 `ERROR 2003 (HY000): Can't connect to MySQL server on` 的异常
2. 这是因为服务器中 MySQL 对应的端口号 **3306** 没有开放
3. 如果使用的是阿里云 ECS 服务器，可以参考以下配置
4. [阿里云 ECS 服务器配置对外开放的端口号](http://asing1elife.com/linux/aliyun/2019/06/18/阿里云-ECS-服务器配置对外开放的端口号/)