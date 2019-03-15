---
title: SpringMVC + Shiro 集成 CAS
date: 2019-03-15
author: asing1elife
categories:
 - java
 - shiro
 - cas
tags:
 - java
 - shiro
 - cas
---

> shiro 在 1.2 版本之后加入 shiro-cas 支持 sso 的 cas 登录验证，以下给出具体的对接方式  

## 相关网址
1. [从这里知道了 shiro.xml 的具体配置](https://blog.csdn.net/qq_25223941/article/details/78316108)
2. [讲解了最基础的通过 Filter 控制 CAS](https://blog.csdn.net/qq_34021712/article/details/81318649)
3. [可能会碰到的重定向问题](https://blog.csdn.net/yy251066394/article/details/78748681)
4. [如何搭建 CAS Server](https://blog.csdn.net/u011872945/article/details/81047025)
5. [GitHub - apereo/cas-overlay-template at 5.3](https://github.com/apereo/cas-overlay-template/tree/5.3)



## 写在前面的话

1. **CAS (Central Authentication Service)** 是实现 **SSO (Single Sign On) [单点登录]** 的一个框架。还有其他框架，例如 Oauth
2. SSO 的目的是实现多个应用系统共用一套登录行为，在 **Session 相同的前提下 [同一个浏览器]** ，用户进入不同系统只需要登录一次



## 搭建 CAS Server

1. 此处搭建 CAS Server 的原因并不是要实现 **从客户端请求到服务器认证** 的全套逻辑，只是因为 **新项目要接入到已经成型的 SSO 体系中**
2. 但是作为新项目在接入客户的 SSO 体系时，很可能客户不怎么配合工作（没错，这就是我碰到的情况），更不可能提供测试环境（没错，别说测试环境，到写这篇笔记时，连生产环境的授权都没通过）
3. 所以对于之前没有写过 SSO 对接的菜鸡（我自己），弄一个 CAS Server 作为测试服务器就至关重要了

### 下载 CAS Server 模版项目

1. 如果只是作为测试服务器的话，CAS Server 不需要从零开始搭建服务器项目，直接前往 [GitHub - apereo/cas-overlay-template at 5.3](https://github.com/apereo/cas-overlay-template/tree/5.3) 下载即可
2. 上述给出的链接是 5.3 版本，截止到写笔记时，最新版本是 6.0 
3. 但最新版使用的是 **gradle + jdk11** ，我的项目用的是 JDK7 ，本地环境也只下载了 JDK8 ，所以最后使用的 5.3 版本，使用的是 **maven + jdk8**
4. 至于如何切换版本，看下图
![](http://asing1elife.com/sources/images/ABEBCE29-5E72-4FC9-9376-A8AA56E75040.png)

### 编译运行 CAS Server 模版项目

1. 按照网站中提供的编译方式 `./build.sh run` 在项目根目录执行即可
	* 此处需要注意一点，就算本地环境中已经安装了 maven ，在运行脚本时依旧会尝试下载 ，而且实测非常慢
	* 解决方式是直接通过下载工具下载对应的 [apache-maven-3.5.2-src.zip](https://repository.apache.org/content/repositories/releases/org/apache/maven/apache-maven/3.5.2/apache-maven-3.5.2-src.zip) 丢到项目根目录后，再执行上述脚本，就可以直接下载成功并且编译通过
2. 编译过程比较漫长，需要下载不少依赖包，全部的依赖包下载完毕后，在编译过程中还会抛出各种异常，不用搭理，直接前往 **/target** 目录获取 **cas.war** 即可
3. 将 cas.war 放置到 tomcat 的 webapps 目录下后启动 tomcat ，war 包就会自动解包并运行
4. 通过浏览器访问 `http://127.0.0.1:8090/cas/login` 可以直接进入 CAS Server 的登录界面
	* 默认用户名 **casuser**
	* 默认密码 **Mellon**

### 为 CAS Server 添加 HTTP 许可

1. 服务器默认并不支持 HTTP 请求，需要对配置文件做以下修改
	* 添加 HTTP 许可的原因是因为如果是 HTTPS 的话，需要编译安全证书，这个过于繁琐了，我们的搭建 CAS Server 的目的只是测试对接是否成功，所以没必要搞那么复杂，直接选用 HTTP 即可
2. 首先停止 tomcat，并前往 webapps 目录找到解包后的 **/cas** 项目

#### 修改 application.properties

1. 具体地址 `/cas/WEB-INF/classes/application.properties`
2. 在文件末尾添加以下代码

```properties
cas.tgc.secure=false 
cas.serviceRegistry.initFromJson=true
cas.serviceRegistry.watcherEnabled=true
cas.serviceRegistry.schedule.repeatInterval=120000
cas.serviceRegistry.schedule.startDelay=15000
cas.serviceRegistry.managementType=DEFAULT
cas.serviceRegistry.json.location=classpath:/services
cas.logout.followServiceRedirects=true
```

#### 修改 HTTPSandIMAPS-10000001.json

1. 具体地址 `/cas/WEB-INF/classes/services/HTTPSandIMAPS-10000001.json`
2. 将内容直接替换成以下代码，应该可以看到默认的 **serviceId** 只有 `^(https|imaps)://.*`

```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^(https|imaps|http)://.*",
  "name" : "测试服务器",
  "id" : 10000001,
  "description" : "测试一下CAS连接",
  "evaluationOrder" : 10000,
   "proxyPolicy" : {
    "@class" : "org.jasig.cas.services.RegexMatchingRegisteredServiceProxyPolicy",
    "pattern" : "^(https|imaps|http)://.*"
  }
}
```

#### 再次启动服务查看修改结果

1. 如果修改成功会显示如下页面
	* 	右侧黄色提示是表示没有使用 HTTPS ，直接忽略
	* 右侧第一个蓝色提示是表示没有使用 **LDAP 或 JDBC 连接数据库** ，导致目前用户数据是写死的，直接忽略（因为测试对接就已经足够了）
	* 右侧第二个蓝色提示就是前文中修改 **HTTPSandIMAPS-10000001.json** 文件后生效的结果
![](http://asing1elife.com/sources/images/C82419FC-8E49-4668-8BC3-9AD48D36AACD.png)



## 为等待对接的项目添加 CAS 支持

### 添加 POM 依赖
1. 在 **pom.xml** 中添加以下依赖
	* **shiro-cas** 是 shiro 自 1.2 版本后添加的对 CAS 的官方实现
	* **cas-client-core** 是 CAS 的核心包

```xml
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-cas</artifactId>
  <version>1.2.4</version>
</dependency>
<dependency>
  <groupId>org.jasig.cas.client</groupId>
  <artifactId>cas-client-core</artifactId>
  <version>3.2.1</version>
</dependency>
```

### 编写 ShiroCasRealm

1. 通常我们在使用 shiro 安全框架时，会编写一个 **ShiroDatabaseRealm** ，继承自 **AuthorizingRealm** ，用于在登录时对用户名密码以及权限的自定义验证
2. 现在项目要通过 CAS 实现 SSO ，说明用户名密码的验证已经在 CAS Server 实现，服务端验证通过后返回到项目的是一个验证通过的唯一标识
3. 所以编写一个 **ShiroCasRealm** ，继承自 **CasRealm** ，来完成对 CAS Server 返回数据的验证
4. 以下代码是具体实现逻辑，因为本项目没有权限验证提现，所以 `doGetAuthorizationInfo()` 函数没有重写
5.  `memberService.getMemberByCas(userId)` 是项目接入服务端用户体系的关键步骤
	* 在没有接入之前项目本身有就已经有自己完整的用户体系，**项目内部其他的需求逻辑都是围绕项目自身的用户体系搭建**
	* 所以在接入服务端用户体系时，就需要通过服务端返回的用户唯一标识来创建一份自己的用户，**同时保证自身用户和服务端用户一对一**，类似于平台用户绑定微信账户后可以通过微信扫码直接登录

```java
public class ShiroCasRealm extends CasRealm {
    private MemberServiceImpl memberService;

    public void setMemberService(MemberServiceImpl memberService) {
        this.memberService = memberService;
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 没有权限验证体系，所以直接返回
        return super.doGetAuthorizationInfo(principals);
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        CasToken casToken = (CasToken) token;

        // token为空直接返回，页面会重定向到 Cas Server 登录页，并且携带本项目回调页
        if (token == null) {
            return null;
        }

        // 获取服务端范围的票根
        String ticket = (String) casToken.getCredentials();

        // 票根为空直接返回，页面会重定向到 Cas Server 登录页，并且携带本项目回调页
        if (!StringUtils.hasText(ticket)) {
            return null;
        }

        TicketValidator ticketValidator = ensureTicketValidator();

        try {
            // 票根验证
            Assertion casAssertion = ticketValidator.validate(ticket, getCasService());
            // 获取服务端返回的用户数据
            AttributePrincipal casPrincipal = casAssertion.getPrincipal();

            // 拿到用户唯一标识
            String userId = casPrincipal.getName();

            // 通过唯一标识查询数据库用户表
            // 如果查询到对应用户则直接返回用户数据
            // 如果没有查询到用户数据则向数据库新增用户并返回用户数据
            MemberDTO member = memberService.getMemberByCas(userId);

            // 将获取到的本项目数据库用户包装为 shiro 自身的 principal 存于当前 session 中
            // 之后在整个项目中都可以通过 SecurityUtils.getSubject().getPrincipal() 直接获取到当前用户信息
            List<Object> principals = CollectionUtils.asList(member, casPrincipal.getAttributes());
            PrincipalCollection principalCollection = new SimplePrincipalCollection(principals, getName());

            return new SimpleAuthenticationInfo(principalCollection, ticket);
        } catch (TicketValidationException e) {
            throw new CasAuthenticationException("Unable to validate ticket [" + ticket + "]", e);
        }
    }
}
```

### 改写 applicationContext-shiro.xml

1. 具体到自己的项目时，不一定叫这个名字，反正就是 shiro 的配置文件

#### 调用自定义 Realm

1. **memberService** 是 **ShiroCasRealm** 中调用的 Service 
	* 在 **MemberService.java** 文件中添加 `@Component("memberService")` 实现 Service 在容器加载时直接注入，这样就不需要在显式的通过 **<bean/>** 方式指定
2. **casServerUrlPrefix** 是 CAS Server 的访问地址
	* 此处使用的是本地测试环境，部署生产时替换为真实环境访问地址即可，或者通过 `<beans profile="dev">` 写两套配置
3. **casService** 是 CAS Server 登录成功后回到本项目的回调地址
	* 必须与后续的 **loginUrl** 中的后半段保持一直，否则会被服务端认为回调不匹配
	* 此处使用的同样是本地测试环境，部署生产时需要替换为真实环境地址

```xml
<bean id="casRealm" class="com.innovaee.ppts.common.security.ShiroCasRealm">
  <property name="memberService" ref="memberService"/>
  <property name="casServerUrlPrefix" value="http://127.0.0.1:8090/cas"/>
  <property name="casService" value="http://127.0.0.1:8080/sop/login"/>
</bean>
```

#### 配置 SessionManager 会话管理器

1. **shiroSessionDAO** 是默认用于缓存 Session 的配置
2. **shiroSimpleCookie** 是默认用户保存 Cookie 的配置
	* **SHAREJSESSIONID** 是重写了默认的 **JSESSIONID** 名称
	* **maxAge** 赋值为 -1 是因为 **实现单点登录后项目本身应该不缓存用户信息**，CAS Server 用户退出后，项目本身的用户信息直接丢失
3. **sessionManager** 是默认的会话管理器
	* **globalSessionTimeout** 赋值为 -1 是因为 **实现单点登录后项目本身应该不限制用户 Session 存放时间** ，项目的 Session 直接从 CAS Server 获取
	* **sessionValidationSchedulerEnabled** 赋值为 true ，表示依旧验证 Session 有效性

```xml
<bean id="shiroSessionDAO" class="org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO"/>

<bean id="shiroSimpleCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
  <constructor-arg name="name" value="SHAREJSESSIONID"/>
  <property name="maxAge" value="-1"/>
</bean>

<bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
  <property name="globalSessionTimeout" value="-1"/>
  <property name="sessionDAO" ref="shiroSessionDAO"/>
  <property name="sessionIdCookie" ref="shiroSimpleCookie"/>
  <property name="sessionValidationSchedulerEnabled" value="true"/>
</bean>
```

#### 配置 SecurityManager 安全管理器

1. **casSubjectFactory** 是默认的工厂类
2. **shiroCacheManager** 是默认的缓存管理器
3. **securityManager** 是默认的安全管理器
	* **realm** 指定为前文中编写的 **casRealm**

```xml
<bean id="casSubjectFactory" class="org.apache.shiro.cas.CasSubjectFactory"/>

<bean id="shiroCacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager"/>

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
  <property name="realm" ref="casRealm"/>
  <property name="sessionManager" ref="sessionManager"/>
  <property name="cacheManager" ref="shiroCacheManager"/>
  <property name="subjectFactory" ref="casSubjectFactory"/>
</bean>
```

#### 配置 CasFilter 登录过滤器

1. **casFilter** 是 shiro 官方实现的 CAS 登录规则过滤器，我们只需要调用并填写失败与成功的回调地址即可
	* **failureUrl** 表示登录失败后会返回到 CAS Server 登录页，同时携带再次登录成功后的本项目登录页
	* **successUrl** 表示登录成功后访问本项目的根目录

```xml
<bean id="casFilter" class="org.apache.shiro.cas.CasFilter">
  <property name="failureUrl" value="http://127.0.0.1:8090/cas/login?service=http://127.0.0.1:8080/sop/login"/>
  <property name="successUrl" value="/app/home"/>
</bean>
```

#### 配置 LogoutFilter 登出过滤器

1. **logoutFilter** 是 shiro 官方实现的 CAS 登出规则过滤器，只需要调用并填写重定向的回调地址即可
	* **redirectUrl** 表示用户在本项目中执行登出操作后，**会重定向到 CAS Server 的登出页**，同时携带再次登录成功后的本项目登录页

```xml
<bean id="logoutFilter" class="org.apache.shiro.web.filter.authc.LogoutFilter">
  <property name="redirectUrl" value="http://127.0.0.1:8090/cas/logout?service=http://127.0.0.1:8080/sop/login"/>
</bean>
```

#### 配置 ShiroFilter 通用过滤器

1. **loginUrl** 是本项目初次访问时会被重定向到 CAS Server 登录页，同时在参数中通过 `service=http://127.0.0.1:8080/sop/login` 指定登录成功后回到本页面的回到地址
	*  `service` 中指定的地址必须与之前 **casRealm** 中指定的 **casService** 保持一致，否则会被服务端认为回调不匹配
2. **filters** 中分别指定了 **logoutFilter** 和 **casFilter** 映射的别名，会在后续请求映射规则中中使用
3. **filterChainDefinitions** 中指定了各种请求会进入哪些过滤器
	* 此处的 `/login = cas` 非常关键，正是因为此处标明只有 `/login` 请求会进入 **casFilter** 
	* 所以在上述所有的 CAS Server 登录成功后回到本项目的回调地址中都携带了 `/login` 请求
	* 这并不是因为本项目需要再次进入登录页面进行登录，而是因为需要通过 **casFilter** 进行一次登录规则验证
	* **如果项目提供给 CAS Server 的回调地址默认不会经过 casFilter ，那么在 Cas Server 登录成功后就可以导致重复重定向**

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
  <property name="securityManager" ref="securityManager"/>
  <property name="loginUrl" value="http://127.0.0.1:8090/cas/login?service=http://127.0.0.1:8080/sop/login"/>
  <property name="successUrl" value="/"/>
  <property name="filters">
    <map>
      <entry key="logout" value-ref="logoutFilter"/>
      <entry key="cas" value-ref="casFilter"/>
    </map>
  </property>
  <property name="filterChainDefinitions">
    <value>
      /logout = logout
      /login = cas
      /** = user,perms,roles
    </value>
  </property>
</bean>
```

#### 配置 Shiro 与 Spring 关联项

1. 这个就不解释了

```xml
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor">
  <property name="proxyTargetClass" value="true"/>
</bean>

<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
  <property name="securityManager" ref="securityManager"/>
</bean>
```

## 结语

1. 按照上述操作依次配置后，项目本身就应该通过 CAS 与客户现有的 SSO 体系对接成功
2. 需要提到的是本次通过 CAS 对接 SSO ，由于原始项目已经使用了 shiro 作为安全框架，所有的配置都在 **shiro.xml** 中操作
	* 默认的如果没有使用安全框架，那么 CAS 的配置则是在 **web.xml** 中完成的，那就是另一个故事了，此处不赘述
	* 友情提供一个普通版本通过 **web.xml** 配置的教程 [普通模式通过 CAS 接入 SSO](https://blog.csdn.net/qq_34021712/article/details/81318649)