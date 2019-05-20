---
title: Windows Server 2008 R2 部署系统经验总结
date: 2019-05-14
author: asing1elife
categories:
- windows
tags:
- windows
- mysql
- redis
- tomcat
---
> 这次客户购买公司产品后提供本地化部署的服务器是 windows server 2008 r2 ，一开始得知这个消息还蛮头疼，毕竟之前的客户提供的都是 Linux 的，自己平时开发使用的是 Mac ，对于 Windows 部署产品，心里却是没有底，不过经过一天的实际操作后，一切都还挺顺利，遂总结如下  

## MySQL 安装及配置服务

### 下载对应软件包并安装
1. 前往 [MySQL :: Download MySQL Installer](https://dev.mysql.com/downloads/windows/installer/5.7.html) 下载对应版本的 MySQL 
	* 链接中提供的是 5.7.x 的版本安装包，当前最新版是 8.x ，这种跳跃式的版本真是让人不放心啊，所以还是选择 5.7.x 比较稳妥
	* 为什么选择 **.msi** 格式的安装包版本，而不是压缩包版本，因为安装包更傻瓜式一点啊，我们要充分发挥 win 的优势啊
2. 如下图所示，第一个是在线安装包，第二个是离线安装包，自然是选择第二个
	* 需要注意的是，链接中显示 **x86, 32-bit** 是指安装包的版本，而不是数据库的版本，这个不用担心，直接下载按照步骤安装即可
![](http://asing1elife.com/sources/images/096016C3-2A48-4BEB-B8F8-C22C05FE0EDF.png)
3. 具体步骤可以参考 [Windows server 2008 r2上安装MySQL - pzczyy - 博客园](https://www.cnblogs.com/pzczyy/p/6289293.html)

### 下载数据库连接软件进行可视化操作
1. Windows 下比较推荐 [Download HeidiSQL](https://www.heidisql.com/download.php) 
2. 下载后安装提示连接即可，具体软件使用方式不赘述了
	* 部署的时候太忙，没时间截图，现在写总结的时候服务器也不再身边，so … 遇到啥操作不熟悉的，就 Google 一下吧
	* 例如 HeidiSQL 查看数据表数据的方式、导入数据的方式，都让我找了一下

### 配置服务自启动
1. 如果按照上述我提供的那个教程操作，在 Windows 服务中的 MySQL 服务，默认就是自启动的
2. 如果不是，可以点击属性自行设置

### 可能涉及到的其他问题

#### HeidiSQL 在导入存储过程中一直抛出 1064 错误
1. 因为 Windows 环境下导入存储过程需要在 SQL 语句最前面添加 `delimiter //`

## Tomcat 安装及配置服务

### 下载对应软件包
1. 前往 [Apache Tomcat® - Apache Tomcat 8 Software Downloads](https://tomcat.apache.org/download-80.cgi) 下载对应版本的 Tomcat
	* 链接中提供的是 8.x 版本，**国际惯例，不用最新的，要用最稳的**
2. 如下图所示，推荐下载 **Core** 中的 **64-bit Windows zip** 版本
	* 这个和 MySQL 推荐安装包版本不一样，Tomcat 选择压缩包版本反而是更大众化，配置方式更统一
![](http://asing1elife.com/sources/images/1C94564D-6A43-4ECB-AC28-C90103F96C1A.png)

### 配置服务自启动
1. 通过 **CMD** 前往 Tomcat 目录下的 **/bin/** 目录
2. 输入 `service.bat install` 即可实现 Tomcat 服务的安装
3. 执行完成后，即可在 Windows 的服务中找到名称为 **Apache Tomcat xxxx** 的服务
	* 按照正常排序，应该就在列表的第一个
4. 服务的启动方式默认不是自启动，需要通过属性进行修改
5. 具体步骤可以参考 [Windows server 2008下配置tomcat到系统服务方法及一般问题解决办法 - Alex_NINE的博客 - CSDN博客](https://blog.csdn.net/Alex_NINE/article/details/75098339)

### 可能涉及到的其他问题

#### 如果需要配置 Spring 的环境变量，应该如何操作
1. Linux 下就是前往 Tomcat 的 **/bin/** 目录，找到 **./catalina.sh** 
2. 在该文件中找到 `JAVA_OPTS` ，将其修改为 `JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=prod"` 即可
3. 理论上在 Windows 中就是找到 **./catalina.bat** 后，在其中找到一样的内容并修改即可
	* BUT ，并没有效果
4. 实际上直接通过计算机右键属性，高级系统配置，环境变量，添加名称为 `spring.profiles.active` 的系统变量，值为 `prod` 即可

## Redis 安装及配置服务

### 下载对应软件包并安装
1. 前往 [Release 3.2.100 · microsoftarchive/redis · GitHub](https://github.com/microsoftarchive/redis/releases/tag/win-3.2.100) 下载对应版本即可
2. 如下图所示，这个依旧推荐下载 **.msi** 格式，可视化安装过程，方便操作
	* 安装过程中有个选项默认没有勾选，意思差不多就是将 Redis 的安装路径添加到系统变量中，记得勾上
![](http://asing1elife.com/sources/images/5A3A174C-87DB-4FDF-A77D-63AC6F87E72F.png)

### 配置服务自启动
1. 在 **CMD** 中输入 `redis-server --service-install redis.windows-service.conf --loglevel verbose` 即可
2. 之后就可以在 Windows 的服务列表中找到名称为 **Redis** 的服务
3. 默认依旧是没有自启动的，需要通过属性自定修改

### 为 Redis 设置密码
1. 前往 Redis 的安装目录，因为上述注册服务时使用的配置文件是 **redis.windows-service.conf** ，所以在安装目录下找到该文件并打开
2. 搜索 `reqiurepass` 之后，可以看到默认是被注释的，在下方添加 `requirepass i_redis` 即可
	* **i_redis** 就是自定义的密码
3. 修改密码并保存后，需要前往服务列表重启服务
4. 在安装目录下双击 **reids-cli.exe** 即可通过命令行连接到 Redis
5. 尝试输入任何语法会提示 `NOAUTH Authentication required` ，则说明需要密码
6. 输入 `auth i_redis` 即可正式连接到数据库