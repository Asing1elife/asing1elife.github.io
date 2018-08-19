---
title: SpringBoot+Shiro前后端分离项目通过JWT实现自动登录
date: 2018-08-09
author: asing1elife
categories:
 - java
 - springboot
 - shiro
 - vue
tags:
 - java
 - springboot
 - shiro
 - vue
---
> 虽然 Shiro 本身可以支持扩展 RememberMe 功能，但仅限于传统项目  
> 因为 Shiro 的用户信息是基于 Session 进行管理，在前后端分离的项目中无法实现 Session 状态的前后统一  
> 所以本文通过 JWT 对 Shiro 原生的 Session 控制进行替换，从而实现用户信息的前后传递及判断  

## 涉及资料
1. [一个已经实现的例子](https://github.com/Smith-Cruise/Spring-Boot-Shiro)
2. [JWT官网](https://jwt.io)
3. [JWT源码](https://github.com/auth0/java-jwt)

## 导入项目所需的依赖
1. 对于 SpringBoot 和 Shiro 的依赖此处不重复介绍
2. 以下是 JWT 的依赖包，就一个即可

```xml
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>3.2.0</version>
</dependency>
```

## 创建 JWTToken 替换 Shiro 原生 Token
1. Shiro 原生的 Token 中存在用户名和密码以及其他信息 [验证码，记住我]
2. 在 JWT 的 Token 中因为**已将用户名和密码通过加密处理整合到一个加密串中**，所以只需要一个 token 字段即可

```java
public class JWTToken implements AuthenticationToken {

    // 密钥
    private String token;

    public JWTToken(String token) {
        this.token = token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```

## 创建 JWTUtil 整合 JWT 相关操作
1. 在这个工具类中可以实现**对用户名和密码的加密处理**，**校验 token 是否正确**，**获取用户名等操作**
2. `Algorithm algorithm = Algorithm.HMAC256(password)` 是对密码进行加密后再与用户名混淆在一起
3. 在签名时可以通过 `.withExpiresAt(date)` 指定 token 的过期时间

```java
public class JWTUtil {

    // 过期时间30天
    private static final long EXPIRE_TIME = 24 * 60 * 30 * 1000;

    /**
     * 校验token是否正确
     *
     * @param token    密钥
     * @param username 登录名
     * @param password 密码
     * @return
     */
    public static boolean verify(String token, String username, String password) {
        try {
            Algorithm algorithm = Algorithm.HMAC256(password);

            JWTVerifier verifier = JWT.require(algorithm).withClaim("username", username).build();

            DecodedJWT jwt = verifier.verify(token);

            return true;
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 获取登录名
     *
     * @param token
     * @return
     */
    public static String getUsername(String token) {
        try {
            DecodedJWT jwt = JWT.decode(token);

            return jwt.getClaim("username").asString();
        } catch (JWTDecodeException e) {
            return null;
        }
    }

    /**
     * 生成签名
     *
     * @param username
     * @param password
     * @return
     */
    public static String sign(String username, String password) {
        try {
            // 指定过期时间
            Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);

            Algorithm algorithm = Algorithm.HMAC256(password);

            return JWT.create()
                    .withClaim("username", username)
                    .withExpiresAt(date)
                    .sign(algorithm);
        } catch (UnsupportedEncodingException e) {
            return null;
        }
    }

}
```

## 创建 JWTFilter 实现前端请求统一拦截及处理
1. `executeLogin()` 方法中的 `getSubject(request, response).login(token)` 就是触发 **Shiro Realm** 自身的登录控制，具体内容需要手动实现
2. `executeLogin()` 始终返回 **true** 的原因是因为具体的是否登录成功的判断，需要在 **Realm** 中手动实现，此处不做统一判断
3. `LOGIN_SIGN` 是前端放置在 headers 头文件中的登录标识，如果用户发起的请求是需要登录才能正常返回的，那么头文件中就必须存在该标识并携带有效值

```java
public class JWTFilter extends BasicHttpAuthenticationFilter {

    // 登录标识
    private static String LOGIN_SIGN = "Authorization";

    /**
     * 检测用户是否登录
     * 检测header里面是否包含Authorization字段即可
     *
     * @param request
     * @param response
     * @return
     */
    @Override
    protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
        HttpServletRequest req = (HttpServletRequest) request;

        String authorization = req.getHeader(LOGIN_SIGN);

        return authorization != null;

    }

    @Override
    protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest req = (HttpServletRequest) request;
        String header = req.getHeader(LOGIN_SIGN);

        JWTToken token = new JWTToken(header);

        getSubject(request, response).login(token);

        return true;
    }

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        if (isLoginAttempt(request, response)) {
            try {
                executeLogin(request, response);
            } catch (Exception e) {
                throw new TSharkException("登录权限不足！", e);
            }
        }

        return true;
    }

    /**
     * 对跨域提供支持
     *
     * @param request
     * @param response
     * @return
     * @throws Exception
     */
    @Override
    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;
        httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
        httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,PUT,DELETE");
        httpServletResponse.setHeader("Access-Control-Allow-Headers", httpServletRequest.getHeader("Access-Control-Request-Headers"));
        // 跨域时会首先发送一个option请求，这里我们给option请求直接返回正常状态
        if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
            httpServletResponse.setStatus(HttpStatus.OK.value());
            return false;
        }
        return super.preHandle(request, response);
    }
}
```

## 自定义 ShiroDatabaseRealm 实现 Shiro Realm 的登录控制
1. 重写 **Realm** 的 `supports()`  方法是通过 JWT 进行登录判断的关键
	* 因为前文中创建了 **JWTToken** 用于替换 Shiro 原生 token
	* 所以必须在此方法中显式的进行替换，否则在进行判断时会一直失败
2. 登录的合法验证通常包括 **token 是否有效** 、**用户名是否存在** 、**密码是否正确**
	* 通过 **JWTUtil** 对前端传入的 token 进行处理获取到用户名
	* 然后使用用户名前往数据库获取到用户密码
	* 再将用户面传入 **JWTUtil** 进行验证即可

```java
public class ShiroDatabaseRealm extends AuthorizingRealm {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private ShiroAuthService shiroAuthService;

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        logger.info("doGetAuthorizationInfo+" + principals.toString());

        String username = JWTUtil.getUsername(principals.toString());
        MemberDTO member = shiroAuthService.getPrincipal(username);

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        List<String> userPermissions = shiroAuthService.getPermissions(member.getId());

        // 基于Permission的权限信息
        info.addStringPermissions(userPermissions);

        return info;
    }

    /**
     * 使用JWT替代原生Token
     *
     * @param token
     * @return
     */
    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JWTToken;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
        String token = (String) authcToken.getCredentials();

        String username = JWTUtil.getUsername(token);

        MemberDTO member = shiroAuthService.getPrincipal(username);

        // 用户不会空
        if (member != null) {
            // 判断是否禁用
            if (member.getStatus().equals(MemberStatus.disableStatus)) {
                throw new DisabledAccountException("901");
            }

            // 密码验证
            if (!JWTUtil.verify(token, username, member.getLoginPassword())) {
                throw new UnknownAccountException("900");
            }

            return new SimpleAuthenticationInfo(token, token, "realm");
        } else {
            throw new UnknownAccountException("900");
        }
    }

}
```

## 在 ShiroConfiguration 中将所有的请求指向 JWT
1. 指定手工实现的 **ShiroDatabaseRealm** 用于传入 **DefaultWebSecurityManager**
2. 在 **securityManager** 中关闭默认的 Session 控制
	* 因为在前后分离项目中前端是无法获取到后端 Session 的，即无法实现用户登录状态的同步
3. 在 `shiroFilterFactoryBean()` 中传入自定义的 **TokenFilter**
	* 并将所有的请求指向该过滤器 `filterRuleMap.put("/**", "jwt")`

```java
@Configuration
@ConditionalOnWebApplication
public class ShiroConfiguration {

    @Bean
    public ShiroDatabaseRealm shiroDatabaseRealm() {
        return new ShiroDatabaseRealm();
    }

    @Bean("securityManager")
    public DefaultWebSecurityManager securityManager(ShiroDatabaseRealm shiroDatabaseRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(shiroDatabaseRealm);

        // 关闭自带session
        DefaultSessionStorageEvaluator evaluator = new DefaultSessionStorageEvaluator();
        evaluator.setSessionStorageEnabled(false);

        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        subjectDAO.setSessionStorageEvaluator(evaluator);

        securityManager.setSubjectDAO(subjectDAO);

        return securityManager;
    }

    @Bean("shiroFilter")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();

        Map<String, Filter> filterMap = new HashMap<>();
        filterMap.put("jwt", new JWTFilter());

        factoryBean.setFilters(filterMap);
        factoryBean.setSecurityManager(securityManager);

        Map<String, String> filterRuleMap = new HashMap<>();

        filterRuleMap.put("/**", "jwt");

        factoryBean.setFilterChainDefinitionMap(filterRuleMap);

        return factoryBean;
    }

    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        defaultAdvisorAutoProxyCreator.setProxyTargetClass(true);
        return defaultAdvisorAutoProxyCreator;
    }

    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(DefaultWebSecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
}
```

## 执行登录操作时对用户信息进行基础验证并签名返回
1. `ResponseData` 是前后分离项目中用于统一后端返回数据的类，必不可少
	* 一般包含状态值和返回的具体内容

```java
@RestController
@RequestMapping("")
@Api("用户登录")
public class LoginController extends AbstractBaseController {

    @Autowired
    private LoginServiceImpl loginService;

    @PostMapping("/login")
    @ApiOperation("登录")
    public ResponseData login(@RequestParam String username, @RequestParam String password) {
        return new SimpleActionHandler(request) {
            @Override
            public void doAction(ResponseData responseData) throws Exception {
                responseData.setData(loginService.jwtLogin(username, password));
            }
        }.handle();
    }

}
```

1. 进行登录操作处理时需要判断数据有效性，即 **数据是否为空**，**密码是否正确**
2. 一般为了用户隐私和安全起见，数据库中存入的密码都是经过 **不可逆混淆** 处理的
	* 所以此处在通过 **JWTUtil** 签名之前，需要将用户传入的密码进行相同混淆，再将混淆后的两个密码进行对比
	* 如果密码正确，通过相同规则混淆后的密码也会相同

```java
@Service
public class LoginServiceImpl extends AbstractBaseService {

    @Autowired
    private MemberServiceImpl memberService;

    /**
     * 用户登录
     *
     * @param username
     * @param password
     * @return
     */
    public String jwtLogin(String username, String password) {
        Assert.notNull(username, "用户名不能为空");
        Assert.notNull(password, "密码不能为空");

        // 获取用户密码混淆值
        MemberDTO member = memberService.getMemberSalt(username);

        // 加密当前用户输入密码
        byte[] bytePassword = DigestUtil.sha1(password.getBytes(), EncodeUtils.hexDecode(member.getSalt()), Constants.PASSWORD_HASH_INTERATIONS);
        String encodePassword = EncodeUtils.hexEncode(bytePassword);

        if (!encodePassword.equals(member.getLoginPassword())) {
            throw new TSharkException("900");
        }

        return JWTUtil.sign(username, encodePassword);
    }
}
```

## 在前端的请求发起基类中对头文件进行统一传递
1. 由于本项目用的 SPA 模式，除了欢迎页不需要登录，其余页面都需要登录后才能访问
	* 因此在基类中对头文件进行统一处理，默认所有请求都会传递用户登录状态
	* 个别不需要传递登录状态的，再进行单独处理
2. 在头文件中指定 `Authorization` 自定义信息，对应的 token 值是用户登录后存入到 **vuex** 中的数组

```javascript
import axios from 'axios'
import router from 'router'
import store from 'store'
import qs from 'qs'
import { SET_USER_INFO } from 'store/mutation-types'
import * as config from 'assets/scripts/config/config'

axios.defaults.withCredentials = true

const setUserInfo = function (user) {
  store.commit(SET_USER_INFO, user)
}

const getUserInfo = function () {
  return store.state.userInfo.token
}

export default function fetch(options) {
  return new Promise((resolve, reject) => {
    const instance = axios.create({
      baseURL: `${config.serverBaseUrl}mop`,
      timeout: 10000,
      withCredentials: true,
      credentials: 'include'
    })

    // 针对头文件是否为空，会做一些权限控制
    if (options.headers === undefined) {
      // 没有指定头文件的使用默认头文件，需要传入用户信息
      options.headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': getUserInfo()
      }
    }

    // 表单提交格式需要进行参数转换
    if (options.headers['Content-Type'].indexOf('x-www-form') !== -1) {
      // 参数格式转换
      options.data = qs.stringify(options.data)
    }

    instance(options).then((response) => {
      let result = response.data

      // 无数据
      if (!result) {
        return false
      }

      // 未授权
      if (result.status === config.UNAUTHORIZED_CODE || result.data === config.UNAUTHORIZED_CODE) {
        // 清空用户信息
        setUserInfo(null)
        // 跳转至登录界面
        router.replace({name: 'portal'})
        return false
      }

      resolve(result)
      return false
    }).catch((error) => {
      reject(error)

      // 清空用户信息
      setUserInfo(null)

      router.replace({name: 'portal'})
    })
  })
}
```

## 单独处理不需要头文件的请求
1. 由于 **fetch.js** 中对头文件存在 **undefined** 的判断，所以外部如果传入头文件，则可以替换默认的头文件配置

```javascript
import fetch from 'assets/scripts/fetch/fetch'
import * as config from 'assets/scripts/config/config'

let baseUrl = '/api/portals'

/**
 * 轮播列表
 */
export function covers() {
  const url = `${baseUrl}/covers`

  return fetch({
    url: url,
    method: config.GET,
    headers: config.VISITOR_HEADER
  })
}
```

1. 实际开发中通常会存在一个 **config.js** 文件，用于存储一系列通用配置信息

```javascript
// 游客头文件
export const VISITOR_HEADER = {
  'Content-Type': 'application/x-www-form-urlencoded'
}
```

## 用户登录成功后将后端返回的 token 值存入 cookie
1. 登录操作实际应该对用户名和密码进行有效性判断，以及登录失败后的错误提示，此处都省略，重点突出将 token 值存入 **vuex** 的步骤

```javascript
<template>
  <div ref="portal" class="container portal-panel">
	  ...
    <mt-field placeholder="请输入手机号或邮箱" v-model.trim="user.username" :state="state.username" 
                        @keyup.enter.native="sendLogin"></mt-field>
    <mt-field placeholder="请输入密码" type="password" v-model.trim="user.password" :state="state.password"
                        @keyup.enter.native="sendLogin"></mt-field>
    <mt-button type="primary" @click.native="sendCaptcha">{{captchaText}}</mt-button>
	  ...
  </div>
</template>

<script type="text/ecmascript-6">
  import { mapMutations } from 'vuex'
  import { SET_USER_INFO } from 'store/mutation-types'

  export default {
    name: 'portal',
    data() {
      return {
        countDown: 0,
        timer: null,
        user: {
          username: '',
          password: ''
        }
      }
    },
    methods: {
      ...mapMutations({
        set_user_info: SET_USER_INFO
      }),
      // 用户登录
      sendLogin() {
        ...
        this.$fetch.login.login({
          username: this.user.username,
          password: this.user.password
        }).then((res) => {
			...
          this.$toast.success('登录成功')

          this.set_user_info({
            token: res.data,
            isLogin: true
          })
			...
        })
      }
		...
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  ...
</style>
```

## vuex 处理 token 值
1. 在 **mutations.js** 中，当存入用户信息时
	* 一份存入 session ，供日常使用
	* 一份存入 cookie ，有效期 30 天，供自动登录使用，具体的 cookie 存值方式，请参见 [简单封装浏览器 cookie 工具类](http://asing1elife.com/vue/2018/08/10/简单封装浏览器-cookie-工具类/)
2. 同时需要确保，当用户手动退出后执行用户信息清除时，需要同时清除 cookie 

```javascript
/** vuex所有的mutation */

// 引入mutations-types
import * as types from './mutation-types'
import { sessionStorage, cookieStorage } from 'assets/scripts/storage'

// 定义mutation，其内部是一些修改方法
const mutations = {
  // 第一个参数是状态值
  // 第二个参数为提交状态修改是传入的对象参数
  [types.SET_USER_INFO](state, userInfo) {
    state.userInfo = userInfo || {}

    if (userInfo === null) {
      sessionStorage.remove('userInfo')
      cookieStorage.remove('userInfo')
    } else {
      sessionStorage.set('userInfo', userInfo)
      cookieStorage.set('userInfo', userInfo, 30)
    }
  }
	...
}

// 暴露给外部
export default mutations
```

## 在请求首页时判断是否可以自动登录
1. 在 **router.js** 的最后添加 `router.beforeEach()` ，可以在链接跳转之前执行一些预判操作
2. 如果 cookie 中存在用户登录信息，则将信息取出重新放入 **vuex** ，并在后续操作中直接跳转至登录后的首屏页面

```javascript
/**
 * 全局路由配置
 * 路由开始之前的操作
 */
router.beforeEach((to, from, next) => {
  // 获取当前请求的名称
  // let toName = to.name
  // 获取当前请求的路径
  let toPath = to.path

  // 请求首页时判断用户是否存在本地登录信息
  if (toPath.indexOf('portal') === 1) {
    // 获取用户信息
    let userInfo = cookieStorage.get('userInfo')

    // 存在用户信息
    if (userInfo !== undefined && userInfo.token !== undefined) {
      // 将用户信息重新进行状态管理
      store.commit(SET_USER_INFO, userInfo)
    }
  }

  // 获取用户登录标识
  let isLogin = store.state.userInfo.isLogin

  // 用户未登录，且请求的不是首页或首页子页面
  if (!isLogin && toPath.indexOf('portal') !== 1) {
    // 跳转到登录页面
    next({
      name: 'portal'
    })
  } else {
    // 用户已登录，且请求的是登录页面
    if (isLogin && toPath.indexOf('portal') === 1) {
      // 跳转到首页
      next({
        path: serverBaseUrl
      })
    } else {
      // 默认操作
      next()
    }
  }
})
```