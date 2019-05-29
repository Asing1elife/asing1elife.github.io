---
title: MySQL GROUP BY 报错
date: 2017-05-22
author: asing1elife
categories:
- mysql
tags:
- mysql
---
> 使用 GROUP BY 时抛出 only_full_group_by 异常  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 问题
1. 通过 GROUP BY 查询时抛出下列异常

```shell
Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'xxx' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

## 解决
### 当前会话生效
1. 在数据库可视化工具中输入以下脚本
2. 但只能当前会话生效，通过其他会话访问时依旧抛出异常，没有任何效果

```sql
SET GLOBAL sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
```

### 配置全局生效
#### 可以找到 my-defauilt.cnf
1. 一般网上各种关于这个问题的帖子，基本都会提到 **my-default.cnf** 这个文件，这是通过 Homebrew 安装 MySQL 后会在指定目录下生成的默认文件
2. 如果能在 `/usr/local/opt/mysql/support-files/` 目录下找到该文件
3. 则直接运行  `cp /usr/local/opt/mysql/support-files/my-default.cnf /etc/my.cnf` 将该文件复制到 `/etc/` 目录下，并重命名为 **my.cnf**
4. 输入 `vi /etc/my.cnf` ，在文件最末尾添加如下脚本
	* 注意该脚本和第一种解决方式中脚本的区别，等号后面的部分没有被单引号爆过，而且末尾没有分号

```
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```
5. 退出并保存文件后，输入如下脚本重启数据库即可

```sh
brew services stop mysql
brew services start mysql
```

#### 找不到 my-default.cnf
1. 上一个解决方法是我之前碰到问题是记录的，当时确实可以在对应目录找到默认配置文件，但这次换电脑重装了 **8.x** 版本的 MySQL ，在任何目录都找不到该文件
2. 所以才有了以下解决办法，参考自 [For homebrew mysql installs, where’s my.cnf? - Stack Overflow](https://stackoverflow.com/questions/7973927/for-homebrew-mysql-installs-wheres-my-cnf)
3. 在终端输入 `sudo /usr/libexec/locate.updatedb` 
	* 会需要等待一段时间，命令会自动执行完成
4. 之后输入 `locate my.cnf` ，就会在命令下方输出初始化的 **my.cnf** 所在目录，如下图
![](http://asing1elife.com/sources/images/7EA70289-486E-4CF2-A089-1DFC613F1AC5.png)
5. 输入 `vi /usr/local/etc/my.cnf` ，在文件末尾添加上一个解决方法相同的脚本
6. 退出保存后重启数据库即可