---
title: Linux CentOS7 安装 ActiveMQ
date: 2019-09-17
author: asing1elife
categories:
- linux
- activemq
- centos
- java
tags:
- linux
- activemq
- centos
- java
---

> 介绍如何在 Linux CentOS7 中按照 ActiveMQ ，并配置启动  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址

1. [linux activemq安装 - 博客园](https://www.cnblogs.com/wenchengqingfeng/p/9896824.html)
2. [centos7 ActiveMQ设置开机启动 - CSDN博客](https://blog.csdn.net/u012249177/article/details/81322874)

## 安装步骤
### 下载安装包
1. 在终端执行 `wget http://apache.fayea.com/activemq/5.15.9/apache-activemq-5.15.9-bin.tar.gz`
2. 如果对应网址的安装包依旧存在，则可以直接下载到当前目录
3. 如果不存在，则会抛出以下异常
![](http://asing1elife.com/sources/images/3B36DF49-6161-4ACC-906A-0A2F0F3D2B8D.png)
4. 可在浏览器中输入 `http://apache.fayea.com/activemq` 查看当前有哪些版本的安装包可以进行下载
5. 如下图，当前可下载的版本就是 **5.15.9** 和 **5.15.10**
6. 将上述命令的版本号改成与之对应的即可直接在 Linux 中下载对应安装包
![](http://asing1elife.com/sources/images/913F65CD-2126-4605-B0F2-AC2C8AF20B0A.png)

### 解压安装包
1. 下载成功后，在当前目录执行 `tar - zxvf apache-activemq-5.15.9-bin.tar.gz` ，可以解压安装包
2. 之后通常会将解压到的软件移到对应目录
	* 因为一般下载软件都会在 **/tmp/** 目录进行
	* 所以可以执行 `mv /tmp/apache-activemq-5.15.9 /opt/` ，将软件放置在该目录中

### 配置环境变量
1. 执行 `vi /etc/profile` ，在文件末端加入以下脚本

```sh
export ACTIVEMQ_HOME=/opt/apache-activemq-5.15.9
export PATH=$PATH:$ACTIVEMQ_HOME/bin
```
2. 如果 `PATH` 变量已经存在，则在其末端加入 `:$ACTIVEMQ_HOME/bin` 即可
3. 执行 `:wq` 保存并退出当前文件
4. **需要注意的是** ，还要执行 `source /etc/profile` 才能使刚才配置的环境变量生效

### 配置全局启动
1. 前往 **/etc/init.d** ，输入 `vi ./activemq` ，创建并编辑启动脚本
2. 输入以下内容之后，保存并退出

```sh
#!/bin/sh

export ACTIVEMQ_HOME=/opt/apache-activemq-5.15.9

case $1 in
    start)
        sh $ACTIVEMQ_HOME/bin/activemq start
    ;;
    stop)
        sh $ACTIVEMQ_HOME/bin/activemq stop
    ;;
    restart)
        sh $ACTIVEMQ_HOME/bin/activemq stop
        sleep 1
        sh $ACTIVEMQ_HOME/bin/activemq start
    ;;

esac
exit 0
```
3. 之后执行 `service activemq start` 即可启动 ActiveMQ
	* 如果启动成功，但通过 `ps -ef|grep activemq` 却看不到启动进程，说明 MQ 启动失败
	* 可以查看 **/etc/profile** 确认 JDK 的版本是否为 1.8
	* 或者直接在 **/etc/init.d/activemq** 中添加 `export JAVA_HOME=/usr/java/jdk1.8.0_74` ，**当然此处应该是你自己的 JDK 路径**

### 相关命令
1. `service activemq start` 启动
2. `service activemq sotp` 停止
3. `service activemq restart` 重启

## 访问可视化界面
### 登录管理页
1. 在浏览器输入 `http://192.168.8.8:8161/admin` 即可进入管理页面，如下图
	* IP 地址是安装 ActiveMQ 的服务器 IP 地址
	* 端口号 8161 是默认的端口号
	* **需要注意的是** ，第一次进入会需要登录，默认的用户名密码是 `admin/admin`
![](http://asing1elife.com/sources/images/434378AE-6164-4C76-BF0E-6A8BC859FB58.png)

### 配置通信队列
1. 在顶部菜单中选择第二个菜单项 **Queues** ，即可进入通信队列配置页面
2. 在 **Queue Name** 后的输入框中输入需要配置的队列名称，并点击 **Create** 按钮即可完成队列配置
![](http://asing1elife.com/sources/images/23B7D2C1-CD57-4F8F-B8ED-4B1B231E46AC.png)

