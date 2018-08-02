---
title: Hibernate @OneToOne 自动新增和删除
date: 2018-01-12
author: asing1elife
categories:
 - java
 - hibernate
tags:
 - java
 - hibernate
---
> Hibernate创建一对一关系时两张表主表保持一致则可以实现自动新增和删除  

## 主表配置
1. 主表对应类中需要通过**@OneToOne**来表示其与从表的关系

```java
@Entity
@Table(name = "sys_user")
public class User {
    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id = new Long(0);

    @OneToOne(cascade = {CascadeType.ALL}, mappedBy = "user", fetch = FetchType.LAZY)
    @JsonIgnore
    private UserAccount userAccount;

    public UserAccount getUserAccount() {
        return userAccount;
    }

    public void setUserAccount(UserAccount userAccount) {
        this.userAccount = userAccount;
    }
}
```

## 从表配置
1. 从表中的**id**需要通过**@GeneratedValue**和**@GenericGenerator**来指明该值从主表中获取
2. 从表中同时需要通过**@OneToOne**来表示其与主表的关系
    * 在维护与主表关系中需要通过**@PrimaryKeyJoinColumn**来表示两个表是主键关联，不存在多余外键

```java
@Entity
@Table(name = "sys_user_account")
public class UserAccount {
    @Id
    @GeneratedValue(generator = "generator")
    @GenericGenerator(name = "generator", strategy = "foreign", parameters = @Parameter(name = "property", value = "user"))
    private Long id = 0L;

    @OneToOne(fetch = FetchType.LAZY)
    @PrimaryKeyJoinColumn
    private User user;

    public UserAccount() {
    }

    public UserAccount(User user) {
        this.user = user;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

## 保存操作
1. 主表在执行保存操作时，需要将从表注入其中，同时将主表的引入也注入到从表，才能实现两个对象的双向关联
2. 当主表的**id**自增成功时，由于从表保有对主表的引用，所以也可以得到主表的**id**

``` java
public void save(User user) {
    if (user.isNew()) {
        user.setUserAccount(new UserAccount(user));

        super.save(user);
    } else {
        super.save(userModel);
    }
}
```