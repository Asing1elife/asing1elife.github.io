---
title: Linux CentOS 部署 Sonarqube7.8 错误排查
date: 2019-09-17
author: asing1elife
categories:
- java
- sonarqube
- sonar
- linux
tags:
- java
- sonarqube
- sonar
- linux
---
> 解决在 Linux CentOS 中部署 Sonarqube 时碰到的一些异常  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 启动失败，并抛出 Process exited with exit value [es] 异常
1. 进入 **/opt/sonarqube-7.8/bin/linux-x86-64/** 执行 `./sonar.sh console` ，得到如下结果
![](http://asing1elife.com/sources/images/A2D35AEE-3BC9-446A-9899-9A15B025748C.png)
2. 在上图中可以看到 **Process exited with exit value [es]: 1** ，因此前往 **/opt/sonarqube-7.8/logs/** 查看 **es.log** 文件，可以得到如下报错
![](http://asing1elife.com/sources/images/73099D40-60C5-46A2-8DAD-3F95B700F97B.png)
3. 从上述报错中可以知道启动失败的原因是因为 **elasticsearch** 进程无法通过 root 用户启用

### 解决方式
1. 参考自 [Linux安装SonarQube - CSDN博客](https://blog.csdn.net/weixin_40816738/article/details/90111803)
2. 由于 **elasticsearch** 进程无法通过 root 用户启用
3. 所以需要新建一个用户，来专门执行启动命令，依次输入如下脚本即可

```sh
# 创建用户
useradd sonar
# 修改该用户密码
passwd sonar
# 赋予权限
chown -R soanr.soanr /opt/sonarqube-7.8
```
4. 之后通过 `su sonar` 切换到该用户，再执行启动命令即可

## 启动失败，并抛出 max virtual memory areas vm.max_map_count [65530] is too low
1. 执行启动命令，得到如下结果
![](http://asing1elife.com/sources/images/40AB1C26-F0F0-4BBC-8996-E70D60F97F2E.png)
2. 在上图中可以看到 **max virtual memory areas vm.max_map_count [65530] is too low** 的异常信息
3. 意思就是说当前系统配置的虚拟内存为 65530 ，太小了

### 解决方式
1. 参考自 [vm.max_map_count - CSDN博客](https://blog.csdn.net/jiankunking/article/details/65448030)
2. 既然虚拟内存太小，那么将虚拟内存加大即可解决问题
3. 使用 root 用户执行 `vi /etc/sysctl.conf`
4. 在文件末尾添加 `vm.max_map_count=262144`
5. 保存并退出后，执行 `sysctl -p` 使配置生效即可
6. 如果想验证是否生效，可以执行 `sysctl -a|grep vm.max_map_count` ，即可看到如下结果
![](http://asing1elife.com/sources/images/2058AB2B-535E-4696-AE55-8918B229D06C.png)

## 启动失败，并抛出 system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk 异常
1. 执行启动命令，得到如下结果
![](http://asing1elife.com/sources/images/D9FE998D-4101-426A-94BB-285BF9752086.png)
2. 在上图中可以看到 **system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk** 的异常信息
3. 错误的原因在 [启动elasticsearch报错解决 - 简书](https://www.jianshu.com/p/1dafec24d582) 中找到
	* 大意就是需要在 elasticsearch 启动时禁止一些系统参数的检测
4. 但上述文章，以及找到的一大堆文章，都是单独讲解 elasticsearch 如果通过修改配置解决这个问题
5. Sonarqube 依赖的 elasticsearch 在其根目录，对应的配置文件在 **/opt/sonarqube-7.8/elasticsearch/config/elasticsearch.yml**
6. 但按照文中的方式添加 `bootstrap.system_call_filter: false` 后，启动依旧报错，没有任何效果

### 解决方式
1. 最后在 [elasticsearch - SonarQube 6.7 - Stack Overflow](https://stackoverflow.com/questions/47210302/sonarqube-6-7-failed-to-start-because-config-seccomp-not-compiled-into-kernel) 中找到了解决方式
2. 执行 `vi /opt/sonarqube-7.8/conf/sonar.properties`
3. 在该文件中找到以下内容，并如图在最后一行添加 `sonar.search.javaAdditionalOpts=-Dbootstrap.system_call_filter=false` 即可解决问题
![](http://asing1elife.com/sources/images/69F4C199-46D5-495D-A142-F84190719D6A.png)

## 修改 Sonarqube 访问网址后，sonar-scanner 执行失败
1. Sonarqube 默认的访问地址是 `http://localhost:9000`
2. 如果想要修改这个地址，只需要前往 **/opt/sonarqube-7.8/conf/sonar.properties** ，并添加如下内容即可
	* 	找到 **WEB SERVER** 的配置区域
	* 分别自定义 `sonar.web.host=192.168.8.8` 用于指定访问的 IP 地址
	* `sonar.web.context=/sonar` 用于指定访问的根目录
	* `sonar.web.port=7654` 用于指定端口号
3. 保存并退出，之后重启 Sonarqube ，就可以通过 `http://192.168.8.8:7654/sonar` 进行访问
![](http://asing1elife.com/sources/images/467B6759-1CD2-4163-BEA7-C9C9BBC17B7E.png)
4. 但是修改该地址后，发现再次在指定项目中执行 `sonar-scanner` ，就无法成功扫描项目了，并抛出如下异常
![](http://asing1elife.com/sources/images/B68AB5FE-BAA2-4D5E-9539-D1CB54E23C51.png)
5. 从上图中可以看到 **SonarQube server [http://localhost:9000] can not be reached** 的异常信息，以及后续一系列连接失败的报错
6. 这是因为 SonarScanner 本身也会配置默认的 Sonarqube 访问地址，默认就是 `http://localhost:9000`

### 解决方式
1. 那么如果我们修改了 Sonarqube 的访问地址，则还应该前往 SonarScanner 的配置文件进行相应修改
2. 前往 **/opt/sonar-scanner-4.0/conf/sonar-scanner.properties** ，可以看到以下内容
![](http://asing1elife.com/sources/images/18714182-F751-480A-90E6-CD430181A32C.png)
3. 重写 `sonar.host.url=http://192.168.8.8:7654/sonar` ，时该配置文件中的地址与 Sonarqube 中配置的自定义地址一致即可