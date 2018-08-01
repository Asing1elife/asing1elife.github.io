---
title: SpringBoot前后分离项目实现自定义登录拦截
date: 2018-06-24
author: asing1elife
categories:
 - java
 - springboot
 - shiro
tags:
 - java
 - springboot
 - shiro
---
> 通常的 shiro 登录拦截对于 `/login` 操作可设置为 `authc` 模式，但前后分离的项目直接设置会导致无法获取登录信息  

## 自定义登录拦截的实现
1. 要实现自定义的登录拦截是继承 `FormAuthenticationFilter` 接口
2. 对接口中的 `onLoginSuccess` 和 `onLoginFailure` 重写
	* 从而根据登录成功和失败进行不同的操作记录

```java
public class LoginFormAuthenticationFilter extends FormAuthenticationFilter {

  // springBoot中存在与配置类的service类需要提供get/set方法手动收入，如果使用@Autowired会导致报错
    private MemberLoginRecordServiceImpl memberLoginRecordService;

    @Override
    protected boolean onLoginSuccess(AuthenticationToken token, Subject subject, 
ServletRequest request, ServletResponse response) throws Exception {
        // 处理会员登录成功操作
        memberLoginRecordService.doAfterSuccessfullyLogin(request, subject);

        return super.onLoginSuccess(token, subject, request, response);
    }

    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, 
ServletRequest request, ServletResponse response) {
        return super.onLoginFailure(token, e, request, response);
    }

    public MemberLoginRecordServiceImpl getMemberLoginRecordService() {
        return memberLoginRecordService;
    }

    public void setMemberLoginRecordService(MemberLoginRecordServiceImpl memberLoginRecordService) {
        this.memberLoginRecordService = memberLoginRecordService;
    }
}
```

## 将重写的拦截器注入到 shiro 配置中
```java
@Configuration
@ConditionalOnWebApplication
public class ShiroConfiguration extends BaseShiroConfiguration {

  ...

  	// 手动注入自定义拦截器需要用到的service
  	// 注入该service即可，service内部的其他类可实现自动注入
    @Bean
    public MemberLoginRecordServiceImpl memberLoginRecordService() {
        return new MemberLoginRecordServiceImpl();
    }

    public ShiroFilterFactoryBean shiroFilterFactoryBean(DefaultWebSecurityManager securityManager) throws Exception {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);

  		// 指定登录链接
        shiroFilterFactoryBean.setLoginUrl("/login");

        Map<String, Filter> filters = new LinkedHashMap<String, Filter>();
        
  		// 配置自定义表单拦截器
        LoginFormAuthenticationFilter authcFilter = new LoginFormAuthenticationFilter();
        authcFilter.setRememberMeParam("rememberMe");
  		// 注入上述引用的service类
        authcFilter.setMemberLoginRecordService(memberLoginRecordService());
  		// 选择指定拦截器的规则
        filters.put("authc", authcFilter);

        shiroFilterFactoryBean.setFilters(filters);

  		// 配置不同接口规则
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
  		// 登出操作由shiro管理
        filterChainDefinitionMap.put("/logout", "anon");
  		// 以下规则可直接访问
        filterChainDefinitionMap.put("/api/portals/**", "anon");

  		// 以下规则需要认证访问
        filterChainDefinitionMap.put("/login", "authc");
        filterChainDefinitionMap.put("/api/**", "authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);

        return shiroFilterFactoryBean;
    }
}
```

## 以上方式在前后分离项目中会出现的问题
1. 由于项目是前后分离的，前端发起的直接是访问 `loginController` 的请求
2. 该请求由于配置了 `authc` 规则，则需要认证才能访问
3. 所以就导致以上配置方式会直接报错
4. 而如果将 `/login` 请求配置为 `anon` 规则，则会导致登录操作无法被自定义表单拦截器拦截

## 解决方式
1. 重写 `FormAuthenticationFilter` 接口的 `onAccessDenied` 方法
2. 判断当前访问该规则的请求是否为登录请求
1. 如果是登录请求则再次判断是否为登录提交请求 `this.isLoginRequest`
	* 因为正常的登录请求中会存在一个访问登录页面和提交登录表单两个操作
	* 而前后分离项目发起的必然是提交登录表单 `this.isLoginSubmission`
2. 如果不是登录请求则返回 false

```java
@Override
protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
    if (this.isLoginRequest(request, response) && this.isLoginSubmission(request, response)) {
        return true;
    }

    return false;
}
```