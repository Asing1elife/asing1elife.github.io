---
title: OpenOffice 在 Linux 下安装使用
date: 2017-04-11
author: asing1elife
categories:
 - java
 - openoffice
 - linux
tags:
 - java
 - openoffice
 - linux
---
> OpenOffice在linux下如何安装及使用  

## 官网
[OpenOffice](http://www.openoffice.org/download/index.html)

## 安装
1. 根据实际的 Linux 版本决定下载类型
![](http://asing1elife.com/sources/images/B85F77C6-99B2-4033-8003-3E67126C3AAB.png)
![](http://asing1elife.com/sources/images/F2D49BFB-7B0A-4E73-AFEB-FFB0C2E6BA5A.png)
2. 在 tar 包所在目录，输入 `tar -xzvf Apache_OpenOffice_4.1.3_Linux_x86-64_langpack-rpm_zh-CN.tar.gz` 解压下载的 tar 包
3. 接下来出来的文件在同目录下的 `zn-CH`  中，里面包含三个目录 `RPMS` `readmes` `licenses`
4. 进入到 `RPMS` 中，执行 `rpm -ivh *rpm` 安装该目录下所有的 rpm 文件

## 使用
1. 进入默认安装目录 `/opt/openoffice4/program` 中
2. 执行 `/opt/openoffice4/program/soffice "-accept=socket,host=127.0.0.1,port=8100;urp;" -headless -nofirststartwizard &`
3. 在命令的最后输入 `&` 可确保服务在后端运行

## 注意
1. 若执行启动命令时报错 `/opt/openoffice4/program/soffice.bin: error while loading shared libraries: libXext.so.6: cannot open shared object file: No such file or directory` ，则需要安装 `libXext` 依赖包，根据 Linux 版本选择安装类型
	* 执行 `yum install libXext.x86_64` 
	* 在 `/usr/lib64` 或 `/usr/lib` 中找到 `libXext.so.6` 文件，复制到 `/opt/openoffice4/program/` 目录中
	* 对复制过来的文件执行 `chmod 777 libXext.so.6`