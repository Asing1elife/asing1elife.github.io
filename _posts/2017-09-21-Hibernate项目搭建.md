---
title: Hibernate项目搭建
date: 2017-09-21
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
---
> Hibernate 作为一个全自动的持久层[ORM]框架，在项目中引入需要以下步骤  

## 基础步骤
1. 进入 [Hibernate官网](http://hibernate.org/orm/) 下载 Hibernate 源码包
2. 解压源码包后进入下图中的目录，将目录中的所有 jar 包复制到项目的 lib 目录中
![](http://asing1elife.com/sources/images/1AB06CA6-9F3E-43A9-BFCD-12C3A496D884.png)
3. 在项目 src 目录下新建 **hibernate.cfg.xml** 文件
    1. 若实体类使用注解，则需要在最后加入对实体类的映射
    2. 若实体类使用配置文件，则需要在最后引入对应配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC 
"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
  "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
<session-factory>
  <!-- 数据库配置信息 -->
  <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
  <property name="hibernate.connection.url">jdbc:mysql:///asl_dev</property>
  <property name="hibernate.connection.username">root</property>
  <property name="hibernate.connection.password">root</property>

  <!-- Hibernate 方言 -->
  <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
  <!-- 是否显示 sql -->
  <property name="hibernate.show_sql">true</property>
  <!-- 是否格式化 sql -->
  <property name="hibernate.format_sql">true</property>
  
  <!-- 注解实体类映射 -->
  <mapping class="online.shixun.hpeu.model.GoodsModel"/>

  <!-- 配置实体类映射 -->
  <mapping resource="com/qingshixun/model/User.hbm.xml"/>
</session-factory>
</hibernate-configuration>
```
## 制作 Hibernate 工具类
1. 通过该工具类可以便捷的重复调用 Session 对象
2. 但如果涉及到 CUD 操作，则需要另外开启事务，并管理事务的提交或回滚

```java
public class HibernateUtil {

    private static SessionFactory sessionFactory;

    static {
        Configuration configuration = new Configuration().configure();
        StandardServiceRegistryBuilder ssrb = new StandardServiceRegistryBuilder();
        StandardServiceRegistry serviceRegistry = ssrb.applySettings(configuration.getProperties()).build();
        sessionFactory = configuration.buildSessionFactory(serviceRegistry);
    }

    public static Session getSession() {
        return sessionFactory.openSession();
    }

}
```