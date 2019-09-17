---
title: Sonarqube 安装及配置
date: 2019-09-06
author: asing1elife
categories:
- sonar
- java
tags:
- sonar
- sonarqube
---
> Sonarqube 是开源的代码质量检测平台  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [Download | SonarQube](https://www.sonarqube.org/downloads/)
2. [Get Started in Two Minutes Guide | SonarQube Docs](https://docs.sonarqube.org/latest/setup/get-started-2-minutes/)

## 安装方式
1. 在 Mac 上通过 Homebrew 安装，自然是最方便的
2. 在终端输入 `brew search sonar` 即可看到如下结果
	* 输入 `brew info sonarqube` 可查看详细信息
![](http://asing1elife.com/sources/images/8DEE8EF0-0608-4EEA-BB4A-195C0313E8F6.png)
3. 在终端输入 `brew install sonarqube` 即可完成安装
	* **Sonarqube 依赖 JDK11 ，如果没有安装 JDK11 ，即使 sonarqube 安装好了 ，也无法启动** 
![](http://asing1elife.com/sources/images/91B4655A-64C3-469A-930D-15898ED02769.png)

## 错误排查
### 执行 sonar console 无法启动
1. `sonar console` 的启动方式是通过 Homebrew 安装后才有的
2. 默认的启动方式应该是到程序的 **/bin** 目录下找到对应平台的 **sonar.sh** 然后执行启动
3. 执行对应启动脚本，抛出错误，如下图
![](http://asing1elife.com/sources/images/F78423A1-4889-4609-B94E-070CEB4020E3.png)

#### 解决方式
1. 在 [java - SonarQube does not start - Stack Overflow](https://stackoverflow.com/questions/29841138/sonarqube-does-not-start) 找到了解决方式
2. 首先找到 JDK11 的完整路径，如下图
![](http://asing1elife.com/sources/images/C95A62A6-B5B3-4D9E-8615-348B9DB5F8E7.png)
3. 之后前往 Sonarqube 的配置文件目录，如下图
	* 这是通过 Homebrew 安装后的软件目录
![](http://asing1elife.com/sources/images/63FC3481-CE94-40AE-9493-AB46F3CEBC81.png)
4. 输入 `vi ./wrapper.conf` 打开对应配置文件，如下图
	* 	在文件第 4 行应该就能找到 `wrapper.java.command=java` ，**下图是已经修改后的效果**
	* 将之前拿到的 JDK11 完整路径粘贴到这里，**最后需要保留/java** 
![](http://asing1elife.com/sources/images/D32ED64C-DB22-4CC6-82AE-140B16AB14CF.png)
5. 之后在执行 sonar console 就能顺利启动了，如下图
	* 	在浏览器输入 **http://localhost:9000** 即可访问管理页
	*  默认用户名和密码是 **admin / admin**
![](http://asing1elife.com/sources/images/EBBB5DBB-F09A-44DD-939C-3323A47B7415.png)

### 配置 MySQL 数据库后 Sonar 无法启动
1. Sonar 自带的 H2 数据库只能用于测试，所以实际使用的话还是需要对接外部数据库
2. 不过在 **/conf/sonar.properties** 中修改了数据库连接配置后却发现程序一直无法启动
3. 具体报错信息如下
![](http://asing1elife.com/sources/images/CB856CE1-7136-4A9A-8347-722F2E1B7592.png)
4. 前往 **/usr/local/Cellar/sonarqube/7.9.1/libexec/logs** 查看日志记录，在 **web.log** 中可以找到如下信息
	* 意思就是说 Sonarqube 7.9 不支持连接 MySQL 数据库
	* 实际上 7.8 版本都是支持 MySQL 的，但 7.9 突然就不支持了，实属无奈
![](http://asing1elife.com/sources/images/B97273E6-6D12-46E8-ABE3-48EF5FE084D8.png)

#### 解决方式
1. 如果实在要使用 MySQL 数据库，则需要将 Sonarqube 降级到 7.8 或者更低版本
2. Sonarqube 7.x 能够支持 MySQL 的的最新版本是 7.8 ，可惜的是没有在 Homebrew 中找到对应的历史版本
3. 只能通过 [Sonarqube 的历史版本资源库](https://binaries.sonarsource.com/Distribution/sonarqube/) 进行下载

### Sonarqube 7.8 版本连接 MySQL 启动失败
1. 在终端前往 Sonarqube 的启动目录并输入 `./sonar.sh console` ，发现启动失败，如下图
	* 从启动日志中并看不到启动失败的原因
![](http://asing1elife.com/sources/images/A18EFCD9-6808-4029-AF80-BB57251B004A.png)
2. 依旧是前往日志目录查看 **web.log** ，可以找到如下信息
	* 	关键在于 `query_cache_size` 参数，这个参数是 MySQL 8.0 之前才有的参数
![](http://asing1elife.com/sources/images/CB4119E6-16E2-4140-976A-BC27AD19809C.png)
3. 会抛出上述异常的原因是本地使用的数据库正好是 MySQL 8.0 版本
	* 然而就算是最后一个支持连接 MySQL 的 Sonarqube 7.8 版本，都无法连接 MySQL 8.0
	* **不要问我怎么知道的，我从 6.7 一直下载到 7.9 ，在每一个半半的 /conf 配置中看到的 ！！**

#### 解决方式
1. 将 MySQL 的版本降低到 5.7 或者 5.6
2. 首先将当前数据库的数据导出备份，然后输入 `brew services stop mysql` 停止 MySQL 服务
3. 输入 `rm -rf /usr/local/var/mysql` 删除当前版本的 MySQL 配置地址
4. 输入 `brew uninstall mysql` 既可以删除当前安装的 MySQL 8.0 版本
5. 之后参照 [homebrew安装mysql 5.7* - 简书](https://www.jianshu.com/p/04e80809802d) 安装即可

### 扫描项目时抛出 sonar.java.binaries 相关异常
1. 具体报错信息如下
![](http://asing1elife.com/sources/images/461F4DD6-1D8E-4605-A98B-FEB787B06CB4.png)
2. 这是因为对于一些完整结构的项目，在扫描时需要指定编译目录
3. 一般指定方式是在 **sonar.properties** 文件中添加 `sonar.java.binaries=**/target/classes`
4. 如果按照这种方式指定，项目目录中就必须存在此目录，否则就会抛出如下异常
![](http://asing1elife.com/sources/images/A574507D-B530-482E-81DE-9B14F7CE05F4.png)
5. 但是有时候项目没有必要编译，自然就不存在此目录，应该如何解决？

#### 解决方式
1. 在 **sonar.properties** 中添加 `sonar.java.binaries=./` 
2. 表示编译目录就是当前目录，即可分析成功