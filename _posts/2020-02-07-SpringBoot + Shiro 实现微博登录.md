---
title: SpringBoot + Shiro 实现微博登录
date: 2020-02-07
author: asing1elife
categories:
- java
- shiro
- weibo
- auth2
tags:
- java
- shiro
- weibo
- auth2
---
> 介绍在服务端使用 SpringBoot + Shiro ，用户端使用 jQuery 的环境下如何实现网站对接微博登录  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. 网站要接入第三方登录，首先考虑的自然是微博和微信以及 QQ
2. 接入之前需要在对应的开放平台去申请权限，微博的需要去 [新浪微博开放平台](https://open.weibo.com/connect) 
	1. 登录后点击 **立即接入** 就可以开始创建应用了，创建过程不赘述，按操作来就好
3. 至于为什么要先接入微博，因为微信的申请需要 7 个工作日，微博只需要 1 个工作日
	1. 而且，最最关键的是，微博的其实在应用创建好之后就可以进行开发了
	2. 但是微信需要审核通过之后才能拿到对应的接口密钥

## 开发前的准备工作
1. 有的同学可能在翻看文档后一脸懵，想着需要提供给第三方的回调地址必须是线上的，那咱本地开发环境如何测试？
2. 为啥不论是微信，还是微博，在它们的开发文档中都完全没提到测试如何进行
3. 其实我们只需要对本机的 **hosts** 文件做一些修改即可
4. 比如提供给微博的授权回调地址是 `http://asing1elife.com/teamnote/social/weibo/callback`
5. 那么我们只需要在本机的 **hosts** 文件中增加一行 `127.0.0.1 asing1elife.com`
6. 然后把本地开发环境的端口改成 80 ，这样原本需要使用 `http://127.0.0.1:8080/teamnote` 才能访问的网站
7. 现在就可以直接使用 `http://asing1elife.com/teamnote` 进行访问了
8. 接着在这样的域名下进行微博的授权，成功回调后也会优先返回到本地的映射中，就可以完美的进行开发和测试了

### 需要注意的一点
1. 如果修改 **hosts** 文件后，在浏览器中仍然只能通过 `127.0.0.1` 访问网站，无法通过映射的域名访问
2. 可以先在终端尝试 ping 一下域名，如果能够顺利 ping 通，那么进行以下检查
	1. 本机是否有开启 VPN 
	2. 浏览器是否有安装网络代理插件
3. 关掉上述对应的软件即可解决，至少我是这么解决的

## 在微博开放平台创建应用并配置信息
### 创建应用
1. 前往 [新浪微博开放平台](https://open.weibo.com/connect) 登录后点击 **立即接入** 就可以开始创建应用
2. 应用创建好之后不需要点击 **提交审核** ，因为就算提交了也不会通过，驳回理由如下
![](http://asing1elife.com/sources/images/5BB0C653-929E-45F7-8802-3C2F35E3A77A.png)
3. 前面就说过，微博登录的接入流程和微信不一样，微信需要先审核通过才能进行开发，而微博是先开发完成再提交审核

### 配置回调地址
1. 在 **我的应用** 中点击左侧的 **应用信息 - 高级信息** 就可以配置授权的回调地址
2. **授权回调页** 就是用户在扫码页面进行扫码，并在手机点击确认操作后，微博会携带回执参数进行回调的页面请求
3. **取消授权回调页** 就是用户在扫毛页面点击了取消按钮，会跛会进行回调的页面请求
4. 这两个地址都可以尽情的修改，一直改到你满意为止，因为这些修改都是立即生效的
![](http://asing1elife.com/sources/images/FE09BAF7-E8D2-42C6-A526-9C239A41D18A.png)

### 获取应用关键信息
1. 在 **我的应用** 中点击左侧的 **应用信息 - 基本信息** 就可以看到应用的两个关键信息
2. **App Key** 在获取微博的扫码页面是需要携带，在获取微博用户的 AccessToken 时也需要携带
3. **App Secret** 在获取微博用户的 AccessToken 时需要携带
4. 把这两个值先记录下来即可
![](http://asing1elife.com/sources/images/E030F5C8-6804-48E2-A3C3-6B0947BD791A.png)

### 配置应用测试信息
1. 在上图的 **应用信息 - 测试信息** 中就可以对测试用户进行配置
2. 因为应用还没有通过审核，所以出于未上线状态，这个时候并不是随便使用一个微博用户扫码都能获取到信息
3. 只有 **应用的开发者账号** 以及 **在测试信息中添加的测试账号** 才能在扫码后获取到信息
	1. 测试账号最多可以添加 15 个
4. 最简单的办法肯定就是直接使用该应用的开发者账号进行开发测试即可

## 画个图简单介绍流程
1. 大概的开发流程就是下图所示
![](http://asing1elife.com/sources/images/weibo-oauth.png)

## 在请求数据过程中抛出错误代码可以参考官方的文档
1. 在 [授权机制说明 - 微博API](https://open.weibo.com/wiki/%E6%8E%88%E6%9D%83%E6%9C%BA%E5%88%B6%E8%AF%B4%E6%98%8E) 页面的最底部可以看到如下图片
![](http://asing1elife.com/sources/images/8442E5B2-25E3-43DA-942A-FB086A422381.png)

## 服务端生成扫码页面请求
1. 获取扫码页面的接口文档 [Oauth2/authorize - 微博API](https://open.weibo.com/wiki/Oauth2/authorize) 中对参数描述的非常清晰

### 生成页面请求之前做一些准备工作
1. 为了方便服务端组装数据，先将参数进行封装
2. 然后还提供一个将实体类字段转化为请求参数的方法

#### 必选参数 AuthorizeRequest
1. 必选参数只有两个，可以将上文中提到的 **AppKey** ，以及设置的 **授权回调页** 直接赋值在这里

```java
public class AuthorizeRequest {
    // 申请应用时分配的AppKey
    private String client_id = "";

    // 授权回调地址，站外应用需与设置的回调地址一致
    private String redirect_uri = "";

    // 省略 Getter/Setter
}
```
2. 特别是 **redirect_uri** ，如果赋值的内容和开放平台应用设置中不一致，在请求扫码页面时就会出现以下错误页面
![](http://asing1elife.com/sources/images/83BC59C7-516E-471F-8DAE-2EE8EC09000A.png)

#### 可选参数 AuthorizeRequestExt
1. 可选参数首先继承自必选参数
2. 目前这些可选参数没什么用，封装起来只是为了以后方便扩展
3. 比如 `scope` 参数，在用户扫码确认时，想要获取例如邮箱信息、关注列表等权限就是通过该字段进行指定

```java
public class AuthorizeRequestExt extends AuthorizeRequest {
    // 申请scope权限所需参数，可一次申请多个scope权限，用逗号分隔
    // 可参考 https://open.weibo.com/wiki/Scope
    private String scope;

    // 用于保持请求和回调的状态，在回调时，会在Query Parameter中回传该参数
    private String state;

    // 授权页面的终端类型，取值见下面的说明
    // default是web浏览器，可选mobile、wap、client、apponweibo
    private String display;

    // 是否强制用户重新登录，默认false
    private boolean forcelogin;

    // 授权页语言，缺省为中文简体版，en为英文版
    private String language;

    // 省略 Getter/Setter
}
```

#### 将 Java 实体类转换为请求参数的方法
1. 这个方法的目的是为了将上述封装好的参数，转换为可以拼接在请求链接末尾的参数字符串
2. 同时为了保证参数的有效性，在转换时会直接过滤掉空的参数

```java
private String truncateClassToParams(Object obj) {
    // 获取自身
    Class<?> clazz = obj.getClass();
    // 获取父类
    Class<?> superClass = clazz.getSuperclass();

    // 先将自身的字段全部获取
    List<Field> fields = Lists.newArrayList(clazz.getDeclaredFields());

    // 再尝试获取父类字段
    if (superClass != null) {
        fields.addAll(Arrays.asList(superClass.getDeclaredFields()));
    }

    StringBuilder paramsBuilder = new StringBuilder();

    for (Field field : fields) {
        // 允许字段可读取
        field.setAccessible(true);

        String name = field.getName();
        String value = null;

        try {
            // 尝试获取字段的值
            Object valueObj = field.get(obj);

            // 值不为空则获取
            if (valueObj != null) {
                value = valueObj.toString();
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

        // 非空的值就拼接到参数中
        if (value != null) {
            paramsBuilder.append(String.format("%s=%s&", name, value));
        }
    }

    String paramsTemp = paramsBuilder.toString();

    // 添加前缀和去掉多余尾缀
    return String.format("?%s", paramsTemp.substring(0, paramsTemp.length() - 1));
}
```

### 生成扫码页面链接
1. 当用户点击页面上的微博按钮时，通过 JS 向服务端发送获取扫码页面链接的请求
2. 然后将下列代码生成的链接返回到前端即可，前端获取到链接后可以使用 `window.open(url, '_self')` 直接替换当前页面

```java
private static final String WEIBO_AUTHORIZE_URL = "https://api.weibo.com/oauth2/authorize";

public String getAuthorizeUrl() {
    String params = truncateClassToParams(new AuthorizeRequestExt());

    return WEIBO_AUTHORIZE_URL + params;
}
```
3. 如果使用生成的链接成功请求到了微博扫码页面，效果如下
![](http://asing1elife.com/sources/images/DBDCB13B-8D73-4381-AD3A-494C590AA1A7.png)

## 接收微博扫码成功的回调请求
1. 要接收回调请求，首先 Controller 层需要提供接口
2. 然后 Service 层需要处理回执参数，才能继续进行下一步操作

### Controller 层提供接口
1. 接口的请求地址必须要和之前开放平台设置的完全一致，否则就无法正常接收回执

```java
@Controller
@RequestMapping("/social/weibo")
public class SocialWeiboController {
    @Autowire
    private SocialWeiboService service;

    @GetMapping("/callback")
    public String callback(Model model) {
        return service.getWeiboAuthorizeCode(model);
    }
}
```

### Service 层接收参数做一些准备工作
1. 在接收参数之前，照例还是将参数进行简单封装

#### 通用参数 AuthorizeResponse
```java
public class AuthorizeResponse {
    // 如果授权请求中传递了该参数，会回传该参数
    private String state;

    // 省略 Getter/Setter
}
```

#### 成功参数 AuthorizeResponseSuccess
```java
public class AuthorizeResponseSuccess extends AuthorizeResponse {
    // 用于第二步调用oauth2/access_token接口，获取授权后的access token
    private String code;

    // 省略 Getter/Setter
}
```

#### 失败参数 AuthorizeResponseFail
```java
public class AuthorizeResponseFail extends AuthorizeResponse {
    // 错误信息
    private String error;

    // 错误编码
    // 参考文档，页面最底端，https://open.weibo.com/wiki/授权机制说明
    // 如果用户在授权页面点了取消，会返回 access_denied:21330
    private String error_code;

    // 省略 Getter/Setter
}
```

### 按照参数获取情况，进行对应的封装
1. 如果能成功获取到 `code` 参数，说明请求是成功的，否则就是失败的
2. 这个地方利用实体类把成功和失败的回执都接收了，目前看着有点多此一举，但对回执进行规范性的接收肯定是没有坏处的

```java
@Service
public class SocialWeiboService {
    @Autowire
    private HttpServletRequest request;

    public String getWeiboAuthorizeCode(Model model) {
        String code = request.getParameter("code");
        String state = request.getParameter("state");

        if (code == null) {
            String error = request.getParameter("error");
            String error_code = request.getParameter("error_code");

            AuthorizeResponseFail responseFail = new AuthorizeResponseFail();
            responseFail.setState(state);
            responseFail.setError(error);
            responseFail.setError_code(error_code);

            return responseFail;
        }

        AuthorizeResponseSuccess responseSuccess = new AuthorizeResponseSuccess();
        responseSuccess.setState(state);
        responseSuccess.setCode(code);

        return responseSuccess;
    }
}
```

### 根据回执参数进行不同的操作
1. 这里如果觉得通过多态来处理过于麻烦，也可以省略上一个方法，直接使用 `request.getParameter("code")` 是否能获取到内容来判断扫码是否成功

```java
@Service
public class SocialWeiboService {
    public String gerWeiboUser() {
        // 这就是上一步骤中封装扫码回执参数的方法
        AuthorizeResponse authorizeResponse = getAuthorizeResponse();

        // 根据回执的类型判断回执是否成功
        if (authorizeResponse instanceof AuthorizeResponseSuccess) {
            // 如果成功则可以进行下一步操作

            // 如果用户登录成功，就重定向到网站登录之后的首页
            return "";
        } else {
            // 失败的处理方式一般就直接返回首页好了
            return "redirect:/index";
        }
    }
}
```

## 通过请求回执获取用户凭证
1. 与该步骤对应的接口文档 [Oauth2/access token - 微博API](https://open.weibo.com/wiki/Oauth2/access_token)
2. 在上述步骤中通过扫码拿到的关键回执参数是 `code` ，但是这个参数本质上的用处是为了下一步用来获取用户凭证
3. 我们切记不能把这个 `code` 作为微博用户的凭证，`code` 本身只是一个临时回执，时效性非常短

### 先做一些获取之前的准备工作
1. 依旧是封装请求和回执参数
2. 以及准备一个用于发送 HTTP 请求的方法

#### 凭证请求参数 AccessTokenRequest
1. 这里继承了上文中的 `AuthorizeRequest` ，因为参数是通用的

```java
public class AccessTokenRequest extends AuthorizeRequest {

    // 申请应用时分配的AppSecret
    private String client_secret = "";

    // 请求的类型
    private String grant_type = "authorization_code";

    // 调用authorize获得的code值
    private String code;

    // 省略 Getter/Setter
}
```

#### 凭证回执参数 AccessTokenResponse
```java
public class AccessTokenResponse {

    // 用户授权的唯一票据，用于调用微博的开放接口，
    // 同时也是第三方应用验证微博用户登录的唯一票据，
    // 第三方应用应该用该票据和自己应用内的用户建立唯一影射关系，来识别登录状态，
    // 不能使用本返回值里的UID字段来做登录识别
    private String access_token;

    // access_token的生命周期，单位是秒数
    private String expires_in;

    // 授权用户的UID，本字段只是为了方便开发者，减少一次user/show接口调用而返回的
    // 第三方应用不能用此字段作为用户登录状态的识别，只有access_token才是用户授权的唯一票据
    private String uid;

    // 省略 Getter/Setter
}
```

#### 从服务端发送 HTTP 请求
```java
private String getHttpRequest(String url, String... methodType) {
    HttpClient httpClient = new HttpClient();

    HttpMethod method;

    // 默认使用 Get
    if (methodType.length != 0) {
        method = new PostMethod(url);
    } else {
        method = new GetMethod(url);
    }

    // 失败后尝试重连3次
    method.getParams().setParameter(HttpMethodParams.RETRY_HANDLER, new DefaultHttpMethodRetryHandler());

    String result = "";

    try {
        int statusCode = httpClient.executeMethod(method);

        result = method.getResponseBodyAsString();

        if (statusCode != 200) {
            throw new Exception("数据获取失败，statusCode -> " + statusCode);
        }

        result = method.getResponseBodyAsString();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        method.releaseConnection();
    }

    return result;
}
```

#### 将请求回执内容从 JSON 转化为实体类
```java
import com.fasterxml.jackson.annotation.JsonInclude.Include;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.util.JSONPObject;
import org.apache.commons.lang3.StringUtils;

public class JsonMapperUtil {

	private static Logger logger = LoggerFactory.getLogger(JsonMapperUtil.class);

	private ObjectMapper mapper;

	public JsonMapperUtil() {
		this(Include.ALWAYS);
	}

	public JsonMapperUtil(Include include) {
		mapper = new ObjectMapper();
		// 设置输出时包含属性的风格
		mapper.setSerializationInclusion(include);
		// 设置输入时忽略在JSON字符串中存在但Java对象实际没有的属性
		mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
	}

  public <T> T fromJson(String jsonString, Class<T> clazz) {
	  if (StringUtils.isEmpty(jsonString)) {
		return null;
	  }

	  try {
		  return mapper.readValue(jsonString, clazz);
    } catch (IOException e) {
		  return null;
	  }
  }
}
```

### 获取 AccessToken
```java
private static final String WEIBO_ACCESS_TOKEN_URL = "https://api.weibo.com/oauth2/access_token";

private AccessTokenResponse getAccessToken(String code) {
    AccessTokenRequest accessTokenRequest = new AccessTokenRequest();
    accessTokenRequest.setCode(code);

    String params = truncateClassToParams(accessTokenRequest);
    String url = WEIBO_ACCESS_TOKEN_URL + params;

    String result = getHttpRequest(url, "POST");

    return new JsonMapperUtil().fromJson(result, AccessTokenResponse.class);
}
```

## 根据获取到的用户凭证进行后续操作
1. 获取到微博用户的用户凭证后，就可以使用该凭证到数据库查询是否存在关联了该凭证的本地用户
2. 如果存在本地用户，则说明该微博用户已经登录过本网站，这个时候就直接登录即可
3. 如果不存在，则稍微麻烦一点，需要继续通过该凭证，以及和凭证一起返回的 UID ，再次向微博发起请求，查询微博用户的个人信息，用于注册本地用户

### 存在本地用户，如何实现直接登录
1. 当本地用户存在时，如何让 Shiro 实现免密登录就是关键所在，因为 Shiro 默认是需要在自定义的 Realm 中对用户名和密码进行匹配的
2. 但用户通过扫码登录的时很显然是没有输入密码的，同时我们也不可能从数据库直接获取用户密码，因为正常的企业级应用中，用户密码一定都是通过加密后再存入到数据库的，网站无法直接获取用户的明文密码
3. 参考自 [shiro实现免密登录，解决三方登录问题 - CSDN博客](https://blog.csdn.net/qq_40194399/article/details/85539158)

#### 重写 UsernamePasswordToken 用于传入登录模式
```java
public class CustomUsernamePasswordToken extends UsernamePasswordToken {
    private String mode;

    // 省略 Getter/Setter
}
```

#### 重写HashedCredentialsMatcher 让 Shiro 支持免密登录
1. 重写 `HashedCredentialsMatcher` 的 `doCredentialsMatch` 方法，从自定义的 Token 中获取传入的登录模式
2. 当登录模式是微博时，就直接返回 TRUE ，表示不再进行其他验证

```java
public class DefaultHashedCredentialsMatcher extends HashedCredentialsMatcher {

    private static final String LOGIN_WEIBO = "weibo";

    @Override
    public boolean doCredentialsMatch(AuthenticationToken authenticationToken, AuthenticationInfo info) {
        CustomUsernamePasswordToken token = (CustomUsernamePasswordToken) authenticationToken;

        if (token.getMode().equals(LOGIN_WEIBO)) {
            return true;
        }

        return super.doCredentialsMatch(token, info);
    }
}
```

#### 在 Shiro 配置中注入重写的 HashedCredentialsMatcher
1. 因为对 `HashedCredentialsMatcher` 进行了重写，所以肯定需要重新注入一下，才能让重写后的 `DefaultHashedCredentialsMatcher` 生效

```java
@Bean
public HashedCredentialsMatcher credentialsMatcher() {
    return new DefaultHashedCredentialsMatcher();
}

@Bean
@DependsOn(value = {"credentialsMatcher"})
public ShiroDatabaseRealm shiroRealm(CredentialsMatcher credentialsMatcher) {
    ShiroDatabaseRealm shiroRealm = new ShiroDatabaseRealm();
    shiroRealm.setCredentialsMatcher(credentialsMatcher);

    return shiroRealm;
}
```

#### 当通过 AccessToken 获取到本地用户时，直接触发 Shiro 登录判定
1. 这里将获取到的 AccessToken 作为用户名，密码是空的，登录类型是微博

```java
private static final String LOGIN_WEIBO = "weibo";

protected void doSocialLogin(String token) {
    CustomUsernamePasswordToken authorizeToken = new CustomUsernamePasswordToken(token, null, LOGIN_WEIBO);

    SecurityUtils.getSubject().login(authorizeToken);
}
```

#### 在重写的 AuthorizingRealm 中再次获取用户
```java
public class CustomShiroRealm extends AuthorizingRealm {

    @Autowired
    private UserAuthService userAuthService;

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken) throws AuthenticationException {
        CustomUsernamePasswordToken token = (CustomUsernamePasswordToken) authcToken;
        token.setRememberMe(true);

        // 获取用户信息
        ShiroPrincipal user = userAuthService.getPrincipal(token.getUsername(), token.getMode());

        // 用户不会空
        if (user != null) {
            // 解码混淆值
            byte[] salt = shiroAuthService.getSalt(user.getSalt());

            return new SimpleAuthenticationInfo(user, user.getPassword(), ByteSource.Util.bytes(salt), getName());
        } else {
            throw new UnknownAccountException("900");
        }
    }
}
```

#### 在 UserAuthService 中根据登录模式采用不同的查询方法
```java
public class UserAuthService {

    private static final String LOGIN_WEIBO = "weibo"; 

    @Autowired
    private UserService userService;

    public ShiroPrincipal getPrincipal(String name, String mode) {
        if (mode.equals(LOGIN_WEIBO)) {
            return userService.getUserByWeibo(name);
        }

        return userService.getUser(name, mode);
    }
}
```

### 不存在本地用户，如何获取微博用户信息
1. 与该步骤对应的接口文档 [/users/show - 微博API](https://open.weibo.com/wiki/2/users/show)
2. 如果不存在本地用户，为了能够通过微博用户直接创建本地用户，只获取到 AccessToken 肯定是不够的
3. 最好是能够获取到微博用户的昵称和头像，这样用户体验才会好一些

#### 获取信息之前封装数据 WeiboUser
1. 获取到的个人信息中会返回很多数据，这里只列举两个有用的

```java
public class WeiboUser {
    // 昵称
    private String screen_name;

    // 头像，一个图片的在线链接
    private String avatar_large;

    // 省略 Getter/Setter
}
```

#### 获取微博用户信息
```java
private final static String GET_WEIBO_USER_URL = "https://api.weibo.com/2/users/show.json?access_token=%s&uid=%s";

public static WeiboUser getWeiboUser(String accessToken, String uId) {
    String url = String.format(GET_WEIBO_USER_URL, accessToken, uId);

    String userInfoJson = getHttpRequest(url);

    return new JsonMapperUtil().fromJson(userInfoJson, WeiboUser.class);
}
```

#### 根据微博用户信息创建用户并直接登录
1. 从微博获取到用户信息后，根据这些信息创建一个本地用户即可
2. **最重要的是不要忘记将之前获取到的 AccessToken 一起保存到数据库的用户信息中**
3. 之后就直接调用上文处理好的 `doSocialLogin` 方法，即可触发 Shiro 的登录判定