---
title: Vue + SpringBoot 集成极验验证码插件
date: 2019-04-16
author: asing1elife
categories:
- vue
- java
- springboot
tags:
- vue
- java
- springboot
---
> 极验有一款行为验证的插件，其实就是个验证码插件，包括滑块和点选的验证方式，这里记录一下如何接入基于 Vue + SpringBoot 的 Web 端项目  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. [jQuery + SpringMVC 集成极验验证码插件](http://asing1elife.com/jquery/springmvc/geetest/2019/04/12/jQuery-+-SpringMVC-%E9%9B%86%E6%88%90%E6%9E%81%E9%AA%8C%E9%AA%8C%E8%AF%81%E7%A0%81%E6%8F%92%E4%BB%B6/)
2. 对插件的具体描述，一些废话，插件相关网址都在上面这篇笔记里
3. 之前这篇是适用于 PC Web 端的，现在这篇是适用于 Mobile Web 端的

## 后端对接实现
1. 后端的具体实现虽然框架不同，一个使用 SpringMVC ，一个则是 SpringBoot ，但具体实现其实没有太大区别
2. Service 层几乎完全一致，唯一的区别就是 Controller 层的请求方式注解不太一样

### 下载并接入集成包
1. 在前面提供的 [geetest-java](https://docs.geetest.com/install/deploy/server/java/) 中可以下载到插件后端环境的集成包，也可以通过 `git clone https://github.com/GeeTeam/gt3-java-sdk.git` 直接下载
2. 下载下来其实是 Java 语言的本地文档、DEMO 、SDK 、以及和后端语言配对的前端依赖 JS
	* 我觉得这里后端如果能提供一个在线的 Maven 或 Gradle 依赖地址其实更好
3. 将 **/src/sdk/GeetestLib.java** 和 **src/demo/demo1/GeetestConfig.java** 引入到自己项目即可
4. 打开 **GeetestConfig.java** ，将其中的 `geetest_id` 和 `geetest_key` 修改成自己的即可
	* 这两个参数需要到他们 [极验后台登录](https://auth.geetest.com/login/) 获取，登录后选择行为验证
	* 之后还需要新增验证，填写基本信息后即可拿到这两个值，如下图
	* 左下角的 ID 对应 `geetest_id` ，KEY 对应 `geetest_key `
![](http://asing1elife.com/sources/images/A96D6B0C-78C4-4BD3-BF74-5D75A488CB21.png)

### 编写验证码服务
1. 后端验证的逻辑分为两步验证，第一步初始化，其实就是相当于生成验证码，第二步才是用户发起验证请求
2. 文档中分别使用了 `doGet` 演示初始化，`doPost` 演示接收验证请求
3. 实际项目中当然不会使用 Servlet 来做这些事情，这里使用的是 SpringMVC
4. 两个操作可以放在同一个 Service 中，例如 **CaptchaServiceImpl.java**

#### 封装请求参数
1. 单独封装起来是因为两步验证都会用到
2. 请求参数其实就是每个用户的唯一标识，文档中使用的是 **用户 ID** 、**终端类型** 以及 **IP 地址**
3. 我这里因为是未登录状态下的验证，所以拿不到用户 ID ，所以直接使用的 IP 地址

```java
private HashMap<String, String> getParams() {
    String clientIp = URIUtil.getClientIp(request);

    HashMap<String, String> params = Maps.newHashMap();
    params.put("user_id", clientIp);
    params.put("client_type", "web");
    params.put("ip_address", clientIp);

    return params;
}
```

#### 生成验证码，对应第一步初始化
1. `getParams()` 获取参数就是上一个函数

```java
public String generateCaptcha() {
    // 初始化极验服务
    GeetestLib lib = new GeetestLib(GeetestConfig.getGeetest_id(), GeetestConfig.getGeetest_key(), GeetestConfig.isnewfailback());

    // 验证预处理
    int status = lib.preProcess(getParams());

    // 将服务状态存放到 Session 中，在第二步验证时会用到
    request.getSession().setAttribute(lib.gtServerStatusSessionKey, status);

    // 返回生成字串
    return lib.getResponseStr();
}
```

#### 接收用户验证请求，并返回存储处理结果
1. `request.getParameter()` 可以直接使用，是因为我的 Service 集成了一个通用的父类，在里面直接注入了 `HttpServletRequest`
2. 通过 `request` 获取的参数都不是自定义的，由极验前端的 **gt.js** 内部提供
3. `saveMessage()` 是验证成功后将当前参与验证的手机号和结果存储到 Redis 中，时效 5 分钟
	* 通过行为验证，正式发起验证码请求时，会先通过请求的手机号去 Redis 中获取行为验证结果
	* 存在且成功才会发验证码，否则会提示进行行为验证
4. `TSharkException()` 是自定义的异常处理，由 Controller 捕获后抛到前端弹出提示框告知用户

```java
public void checkCaptcha(String mobile) {
    // 初始化极验服务
    GeetestLib lib = new GeetestLib(GeetestConfig.getGeetest_id(), GeetestConfig.getGeetest_key(), GeetestConfig.isnewfailback());

    // 接收前端参数，由前端 JS 内部封装处理
    String challenge = request.getParameter(GeetestLib.fn_geetest_challenge);
    String validate = request.getParameter(GeetestLib.fn_geetest_validate);
    String seccode = request.getParameter(GeetestLib.fn_geetest_seccode);

    // 取出第一步初始化验证时存储的服务状态
    int status = (Integer) request.getSession().getAttribute(lib.gtServerStatusSessionKey);

    int result = 0;

    if (status == 1) {
        // 服务器在线
        result = lib.enhencedValidateRequest(challenge, validate, seccode, getParams());
    } else {
        // 服务器离线
        result = lib.failbackValidateRequest(challenge, validate, seccode);
    }

    if (result == 1) {
        // 保存验证信息
        saveMessage(mobile, new MessageBean(mobile, result));
    } else {
        throw new TSharkException("行为验证失败，请检查使用环境");
    }
}
```

### 控制层定义请求
1. 因为是前后分离项目，所以使用 `@RestController` 、`@GetMapping` 以及 `@PostMapping` 来区分请求
2. `SimpleActionHandler` 是自定义封装的 Service 异常处理类
	* `request` 在继承的 `AbstractBaseController` 自定义父类中统一声明
3. `ResponseData` 是自定义封装的返回结果类
4. 后端的服务就完成了，实现起来还是非常简洁的，点个赞

```java
@Api("行为验证")
@RestController
@RequestMapping("/api/captcha")
public class CaptchaController extends AbstractBaseController {

    @Autowired
    private CaptchaServiceImpl captchaService;

    @ApiOperation("生成行为验证")
    @GetMapping("")
    public ResponseData init() {
        return new SimpleActionHandler(request) {
            @Override
            public void doAction(ResponseData responseData) throws Exception {
                responseData.setData(captchaService.generateCaptcha());
            }
        }.handle();
    }

    @ApiOperation("进行行为验证")
    @PostMapping("")
    public ResponseData check(@RequestParam final String mobile) {
        return new SimpleActionHandler(request) {
            @Override
            public void doAction(ResponseData responseData) throws Exception {
                captchaService.checkCaptcha(mobile);
            }
        }.handle();
    }

}
```

## 前端对接实现
1. 前端使用的是 Vue ，但官网文档只提供了普通的 jQuery 版本集成方式

### 前端依赖 JS 在哪里
1. 前端依赖的 JS 在 [geetest-WEB-front](https://docs.geetest.com/install/deploy/client/web) 接口文档中是找不到的，因为不同服务端语言对应的 JS 不一样，所以将其放置在了后端集成包中
2. 具体位置如下图所示
![](http://asing1elife.com/sources/images/50AB49F1-0162-4344-AE61-DF12874ADCF2.png)
3. 官方也给了相关提示，如下图
![](http://asing1elife.com/sources/images/F6329ADA-F2E1-4DFA-AB2E-04333A3A20A2.png)

### 引入前端依赖 JS
1. 将 **gt.js** 直接复制到项目对应的 JS 目录中，然后通过以下方式引入到需要使用的界面

```js
<script type="text/ecmascript-6">
  import 'assets/plugins/geetest/gt'

  export default { ... }
</script>
```

### 准备一个 HTML 区域用于显示行为验证组件
1. 行为验证组件到底放在你自己页面的什么地方，都行，这里只是提供一个样例
2. `mt-cell` 是 **MINT UI** 的组件

```html
<mt-cell class="captcha-wrapper">
  <div class="captcha register-captcha"></div>
</mt-cell>
```

### 初始化行为验证插件
1. 初始化方式其实没有什么变化
2. 我这里加了三个参数传入是因为注册和忘记密码都存在发送验证码的行为，所以都需要行为验证
	* `className` 是行为验证所处的容器
	* `obj` 和 `objCheck` 是注册或忘记密码时填写的信息内容也内容验证状态

```js
export default {
  mounted () {
    this._initGeeTest('register-captcha', this.register, this.rstate)
  },
  methods: {
    // 初始化行为验证
    _initGeeTest (className, obj, objCheck) {
      // 清空之前的历史
      this.captchaStatus = false
      document.getElementsByClassName(className)[0].innerHTML = ''

      this.$fetch.captcha.generate().then((res) => {
        let result = JSON.parse(res.data)

        window.initGeetest({
          gt: result.gt,
          challenge: result.challenge,
          new_captcha: result.new_captcha,
          offline: !result.success,
          product: 'float',
          width: '100%'
        }, (captchaObj) => {
          captchaObj.appendTo(`.${className}`)

          captchaObj.onSuccess(() => {
            // 手机号验证
            if (!this._mobileCheck(obj, objCheck)) {
              captchaObj.reset()
              return false
            }

            let result = captchaObj.getValidate()

            this.$fetch.captcha.check({
              mobile: obj.username,
              geetest_challenge: result.geetest_challenge,
              geetest_validate: result.geetest_validate,
              geetest_seccode: result.geetest_seccode
            }).then((res) => {
              if (!res.success) {
                this.captchaStatus = false
                this.$toast.error(res.data)
                captchaObj.reset()
              } else {
                this.captchaStatus = true
              }
            })
          })
        })
      })
    }
  }
}
```