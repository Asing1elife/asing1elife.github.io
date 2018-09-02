---
title: MySQL GROUP BY 报错
date: 2017-05-22
author: asing1elife
categories:
 - database
 - mysql
tags:
 - database
 - mysql
---
> 使用 GROUP BY 时抛出 only_full_group_by 异常  

## 问题
1. 通过 GROUP BY 查询时抛出下列异常

```
Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'xxx' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

## 解决
1. 设置全局 sql_mode

```
SET GLOBAL sql_mode ='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```
2. 查询设置结果 `SELECT @@GLOBAL.sql_mode;`
3. 在可视化工具中通过 `SET GLOBAL sql_mode` 设置的结果只在会话中生效，数据库重启后则失效
4. 想要设置 sql_mode 全局生效则需要在 mysql 的 my.cnf 文件中填写 sql_mode
5. 通过 homebrew 安装 mysql 后，在 /etc 目录下默认没有创建 my.cnf 文件，需要自行创建 `cp /usr/local/opt/mysql/support-files/my-default.cnf /etc/my.cnf`
6. 进入到 _etc_my.cnf 文件中，在最后输入 sql_mode
7. 重启 mysql