---
title: Linux CentOS7 安装 Nginx
date: 2019-06-18
author: asing1elife
categories:
- linux
- centos
- nginx
tags:
- linux
- centos
- nginx
---
> 介绍如何在 Linux CentOS7 中安装 Nginx  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [CentOS7中使用yum安装Nginx的方法 - 宋兴柱 - 博客园](https://www.cnblogs.com/songxingzhu/p/8568432.html)

## 安装步骤

### 添加 Nginx 源地址
1. CentOS7 默认没有提供 Nginx 的源，但 Nginx 自己提供了，输入如下命令即可

```sh
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

### 安装 Nginx
```sh
sudo yum install -y nginx
```

### 启动 Nginx
```sh
sudo systemctl start nginx.service
```

### 配置 Nginx 开机自启动
```
sudo systemctl enable nginx.service
```

### 尝试登录
1. 输入服务器地址，应该可以直接登录，因为 Nginx 默认的就是 80 端口
2. 如果登录失败，可能是因为阿里云 ECS 服务器对应的 80 端口没有开放
3. 参考 [阿里云 ECS 服务器配置对外开放的端口号](http://asing1elife.com/linux/aliyun/2019/06/18/阿里云-ECS-服务器配置对外开放的端口号/) 即可