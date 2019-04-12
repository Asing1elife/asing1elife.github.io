---
title: SpringMVC+Shiro 实现异步登录
date: 2019-03-07
author: asing1elife
categories:
- java
- springmvc
- shiro
tags:
- java
- springmvc
- shiro
---
> shiro默认的登录方式是传统的页面接收方式，可通过以下方式改成异步发起登录请求并接收登录结果  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 相关网址
1. [CSDN博客](https://blog.csdn.net/u013632755/article/details/51485158)

## 具体实现
1. 本次的处理逻辑是登录成功的请求不做特殊处理，只修改登录失败后的数据返回方式
	* 因为一般登录失败后需要返回具体的失败原因到登录页面
	* 而登录成功后直接跳转到对应的 *successUrl* 即可
### 在 ShiroDatabaseRealm 中对登录结果进行异常捕获
1.  shiro的登录判断是在 `ShiroDatabaseRealm` 中通过 `doGetAuthenticationInfo()` 实现的
	* `ShiroDatabaseRealm`  是用户自定义的 `AuthorizingRealm` 子类，用于处理具体的登录逻辑
2. 在 `ShiroDatabaseRealm` 中做异常捕获的原因是方便后续拦截器直接获取登录失败的原因

```java
public class ShiroDatabaseRealm extends AuthorizingRealm {
    private UniversityMemberServiceImpl universityMemberService;

    public void setUniversityMemberService(UniversityMemberServiceImpl universityMemberService) {
        this.universityMemberService = universityMemberService;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
        UsernamePasswordCheckCodeToken token = (UsernamePasswordCheckCodeToken) authcToken;
        token.setRememberMe(false);

        UniversityMemberDTO member = universityMemberService.getUniversityMember(token.getUsername());

        // 未获取到用户信息可能是因为用户未激活
        if (member == null) {
            throw new UnknownAccountException("用户不存在");
        }

        // 未激活禁止登录
        if (MemberStatus.initStatus.equals(member.getStatus())) {
            // 自定义异常，实现自AccountException
            throw new InactivateAccountException("账号未激活，请激活或联系管理员");
        }

        // 被禁用禁止登录
        if (MemberStatus.disableStatus.equals(member.getStatus())) {
            throw new DisabledAccountException("账号被禁用，请联系管理员");
        }

        byte[] salt = EncodeUtils.hexDecode(member.getSalt());

        return new SimpleAuthenticationInfo(member, member.getLoginPassword(), ByteSource.Util.bytes(salt), getName());
    }
}
```

### 重写 LoginFormAuthenticationFilter
1. 当登录成功或失败后会通过默认拦截器 `FormAuthenticationFilter` 对结果进行拦截
2. 此处的 `LoginFormAuthenticationFilter` 就是继承自 `FormAuthenticationFilter` ，作为自定义的登录拦截器
	* 需要在 `applicationContext-shiro.xml` 中引用 `<entry key="authc" value-ref="loginFormAuthenticationFilter"/>`
3. `isAjax()` 函数用于判断当前请求是否是异步的，避免错误拦截了所有请求
4. 因为登录失败的原因在 `ShiroDatabaseRealm` 中已经指定了，所以大部分原因可以通过 `e.getMessage()` 获取
	* 但是密码是否错误的逻辑判断是在 `ShiroDatabaseRealm` 内部判断的，如果错误会返回 `IncorrectCredentialsException` 异常
	* 此处只需要单独判断异常类型并自定义错误信息即可
5. `onLoginSuccess()` 是登录成功后的拦截
	* 如果不把登录成功后的拦截也修改成异步的，登录成功将会把整个成功后的页面返回到异步登录请求的回调中
6. `onLoginFailure()` 是登录失败后的拦截

```java
public class LoginFormAuthenticationFilter extends FormAuthenticationFilter {
    @Override
    protected boolean onLoginSuccess(AuthenticationToken token, Subject subject, ServletRequest request, ServletResponse response) throws Exception {
        if (!isAjax(request)) {
            issueSuccessRedirect(request, response);
        } else {
            sendResult(response, true, "");
        }

        return false;
    }

    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request, ServletResponse response) {
        // 只拦截ajax请求
        if (!isAjax(request)) {
            setFailureAttribute(request, e);
        }

        String message = "";

        if (e instanceof IncorrectCredentialsException) {
            message = "密码错误";
        } else {
            message = e.getMessage();
        }

        // 发送结果到前端
        sendResult(response, false, message);

        return false;
    }

    /**
     * 异步判断
     */
    private Boolean isAjax(ServletRequest request) {
        return ("XMLHttpRequest").equalsIgnoreCase(((HttpServletRequest) request).getHeader("X-Requested-With"));
    }

    /**
     * 发送结果到前端
     */
    private void sendResult(ServletResponse response, Boolean result, String message) {
        try {
            response.setCharacterEncoding("UTF-8");
            PrintWriter writer = response.getWriter();

            StringBuffer buffer = new StringBuffer();
            buffer.append("{");
            buffer.append("\"success\": ").append(result).append(",");
            buffer.append("\"message\": \"").append(message).append("\"");
            buffer.append("}");

            writer.println(buffer.toString());
            writer.flush();
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```