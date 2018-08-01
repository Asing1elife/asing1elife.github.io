---
title: SpringBoot从数据库加载配置信息
date: 2017-12-31
author: asing1elife
categories:
 - java
 - springboot
 - hibernate
tags:
 - java
 - springboot
 - hibernate
---
> Spring Boot 通过@Value注解可实现获取配置文件中的数据，而配置文件中的数据可以通过修改MutablePropertySources从数据库注入  
> 该示例基于Hibernate实现  

## 实体类
1. 根据Hibernate的配置，实体类对应数据库中的表即可

```java
@Entity
@Table(name = "sys_config")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Config implements Serializable {

    @Id
    private Long id;

    @Column
    private String code;

    @Column
    private String value;

    @Column
    private String name;

    @Column
    private String description;

  ...
}
```

## 服务类
1. 服务类用于从数据库中获取列表信息

```java
@Service
public class SystemConfigService extends SimpleHibernateService<Config, Long> {
   ...
}
```

## 配置类
1. 该配置类会在系统启动时自动加载
2. 根据内部逻辑会将从数据库取出的列表信息注入到**MutablePropertySources**属性集合中
3. 动态注入的属性集合无需有对应的 `.properties` 文件存在

```java
@Configuration
public class SystemConfig {

    @Autowired
    private ConfigurableEnvironment environment;

    @Autowired
    private SystemConfigService service;

    @PostConstruct
    public void initDatabasePropertySourceUsage() {
        // 获取系统属性集合
        MutablePropertySources propertySources = environment.getPropertySources();

        try {
            // 从数据库获取自定义变量列表
            Map<String, String> collect = service.getAll().stream().collect(Collectors.toMap(Config::getCode, Config::getValue));

            // 将转换后的列表加入属性中
            Properties properties = new Properties();
            properties.putAll(collect);

            // 将属性转换为属性集合，并指定名称
            PropertiesPropertySource constants = new PropertiesPropertySource("system-config", properties);

            // 定义寻找属性的正则，该正则为系统默认属性集合的前缀
            Pattern p = Pattern.compile("^applicationConfig.*");

            // 接收系统默认属性集合的名称
            String name = null;
            // 标识是否找到系统默认属性集合
            boolean flag = false;

            // 遍历属性集合
            for (PropertySource<?> source : propertySources) {
                // 正则匹配
                if (p.matcher(source.getName()).matches()) {
                    // 接收名称
                    name = source.getName();
                    // 变更标识
                    flag = true;

                    break;
                }
            }

            if (flag) {
                // 找到则将自定义属性添加到该属性之前
                propertySources.addBefore(name, constants);
            } else {
                // 没找到默认添加到第一位
                propertySources.addFirst(constants);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}
```

## 工具类
1. 由于Spring Boot不支持静态变量的自动注入，所以需要使用一个非静态的**setter**方法将通过**@Value**注解获取到的属性信息赋值给对应静态变量
2. `@DependsOn({"systemConfig"})` 的意思是说 Constants 依赖于 SystemConfig ，所以需要确保 SystemConfig 在 Constants 之前加载

```java
@Configuration
@DependsOn({"systemConfig"})
public class Constants {

    // 资源服务器地址
    public static String RESOURCE_SERVER_URL;

    @Value("${resource.server.url}")
    public void setResourceServerUrl(String resourceServerUrl) {
        RESOURCE_SERVER_URL = resourceServerUrl;
    }
}
```