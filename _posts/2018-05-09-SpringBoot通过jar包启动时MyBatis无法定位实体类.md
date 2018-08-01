---
title: SpringBoot通过jar包启动时MyBatis无法定位实体类
date: 2018-05-09
author: asing1elife
categories:
 - java
 - springboot
 - mybatis
 - error
tags:
 - java
 - springboot
 - mybatis
 - error
---
> SpringBoot 通过 jar 包启动项目时，MyBatis 无法定位实体类，但通过 IDE 启动时没问题  

## 出现问题的原因
1. 通过 jar 启动时，MyBatis 内部获得的路径不同，会导致无法根据配置文件指定的路径扫描到实体类
2. 项目不是通过自动注入方式配置 MyBatis ，而是通过手动注入

## 解决办法
1. 在手动注入并指定实体类扫描路径之前，将 Spring 已经实例化的 VFS 提前指定

```java
@Bean
@ConditionalOnMissingBean(SqlSessionFactoryBean.class)
public SqlSessionFactory sqlSessionFactory(@Qualifier("druidDataSource") DataSource dataSource) throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
  // 指定VFS确保可以扫描到实体类
    sqlSessionFactoryBean.setVfs(SpringBootVFS.class);
    sqlSessionFactoryBean.setTypeAliasesPackage(typeAliasesPackage);

    return sqlSessionFactoryBean.getObject();
}
```