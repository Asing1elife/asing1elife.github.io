---
title: jQuery + SpringMVC 集成极验验证码插件
date: 2019-04-12
author: asing1elife
categories:
- jquery
- springmvc
- geetest
tags:
- jquery
- springmvc
- geetest
---

> 极验有一款行为验证的插件，其实就是个验证码插件，包括滑块和点选的验证方式，这里记录一下如何接入基于 jQuery + SpringMVC 的 Web 端项目  

## 更多精彩
*  更多技术博客，请移步 [IT人才终生实训与职业进阶平台 - 实训在线](https://shixun.online)

## 写在前面的话
1. 接入这个行为验证是因为自己网站的注册验证码短信接口被人攻击了，所以就打算在发送注册验证码之前加一个验证
2. 网上搜索一番看到极验的这个插件，发现还不少公司用的，像经常去的 B 站，和原来偶尔会逛得数字尾巴都有用
3. 这个插件本身效果也还蛮不错的，所以就选择接入
4. 插件本身提供免费试用，只是有几个限制，免费版只提供滑块验证，以及每小时最高 500 次的交互量，咱们这小网站感觉够用了
5. 不过昨天晚上市场那边正好在做一个推广活动，突然给我打电话说用户反馈注册时候无法通过行为验证
6. 我上去一看，行为验证报错了，问了下他们多少人在注册，那边说就 200 个人，我想的是这应该也达不到 500 次的峰值
7. 这得一个人验证错 3 次，才能达到 600 次，所以我就体所应当的以为是后端跪了，但其实没有
8. 后来去极验的后端一看，竟然交互量达到了 636 次，超过了峰值，所以接口返回为空了，汗颜
9. 既然市场这么给力，那我今天就联系一下极验的客服问下升级套餐呗，结果问完之后我就开始着手下线这个功能了
10. 他们的价格差不多是这样的，也不复杂，默认套餐价格区间是 6-20w 一年 ，6w 是支持每小时 1w 次的峰值交互量
11. 我说 1w 都多了，我目前只需要 2000 次，那边给了一个特惠套餐，3w 一年，每小时 5000 次的峰值交互量
12. 我顺便问了一下，这个 3w 一年只仅仅这个插件的授权使用，还是他们几个产品都能用（他们貌似还有其他几个产品，虽然并不感兴趣）
13. 那边说只是验证码，这个我还真的比较汗颜，觉得他们这相当于一个接口卖 3w 一年了，**不过其实应该是我酸，科科**
14. 总之这里只是记录一下我的接入流程，因为接下来我准备删掉这个功能换成别的实现方式了
15. 我现在还在想昨天 200 个人的注册真的可以达到 600+ 次的交互量吗，在有人引导注册的前提下，该不会是 …. ？

## 相关网址
1. [行为验证-Captcha-验证码-Captcha-极验行为验证](https://www.geetest.com/Sensebot)
2. [geetest-java](https://docs.geetest.com/install/deploy/server/java/)
3. [geetest-WEB-front](https://docs.geetest.com/install/deploy/client/web)

## 后端对接实现

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
1. 因为是传统项目（非前后分离），所以使用的注解依旧是以下方式
	* 请求相同，分别使用 `RequestMethod.GET` 和 `RequestMethod.POST` 来区分第一步验证和第二步验证
2. `SimpleActionHandler` 是自定义封装的 Service 异常处理类
	* `request` 在继承的 `AbstractBaseController` 自定义父类中统一声明
3. `ResponseData` 是自定义封装的返回结果类
4. 后端的服务就完成了，实现起来还是非常简洁的，点个赞

```java
@Controller
@RequestMapping("/captcha")
public class CaptchaController extends AbstractBaseController {

    @Autowired
    private CaptchaServiceImpl captchaService;

    @RequestMapping(value = "", method = RequestMethod.GET)
    @ResponseBody
    public ResponseData init() {
        return new SimpleActionHandler(request) {
            @Override
            protected void doHandle(ResponseData responseData) throws Exception {
                responseData.setData(captchaService.generateCaptcha());
            }
        }.handle();
    }

    @RequestMapping(value = "", method = RequestMethod.POST)
    @ResponseBody
    public ResponseData check(@RequestParam final String mobile) {
        return new SimpleActionHandler(request) {
            @Override
            protected void doHandle(ResponseData responseData) throws Exception {
                captchaService.checkCaptcha(mobile);
            }
        }.handle();
    }

}
```

## 前端对接实现

### 前端依赖 JS 在哪里
1. 前端依赖的 JS 在 [geetest-WEB-front](https://docs.geetest.com/install/deploy/client/web) 接口文档中是找不到的，因为不同服务端语言对应的 JS 不一样，所以将其放置在了后端集成包中
2. 具体位置如下图所示
![](http://asing1elife.com/sources/images/50AB49F1-0162-4344-AE61-DF12874ADCF2.png)
3. 官方也给了相关提示，如下图
![](http://asing1elife.com/sources/images/F6329ADA-F2E1-4DFA-AB2E-04333A3A20A2.png)

### 引入前端依赖 JS
1. 将 JS 加入到项目并引入即可
2. 因为我的项目里用到了 jQuery  ，所以以下代码中先引入了

```html
<script type="text/javascript" src="${ctx }/assets/plugins/jquery-1.11.1.min.js"></script>
<script type="text/javascript" src="${ctx }/assets/plugins/geetest/gt.js"></script>
```

### 准备一个 HTML 区域用于显示行为验证组件
1. 具体样式因人而异，总之就是提供一个 `div` 用于放置初始化好的验证组件即可

```html
<section>
  <label class="label">行为验证 <span class="text-danger pull-right"></span></label>
  <label class="input captcha">
    <div class="captcha-tip">行为验证加载中</div>
  </label>
</section>
```

### 初始化行为验证插件
1. 插件的初始化函数是 `initGeetest()` ，来自上文中的 **gt.js**
2. 但其实在初始化之前需要先发送一个异步请求，去调用服务端的初始化接口，因为初始化函数中的值，是由服务端初始化接口返回的
3. 初始化成功后在回调函数中通过 `captchaObj.appendTo()` 将行为验证组件加入指定的页面容器中
4. 因为组件初始化需要先访问后端接口，考虑到网络延迟，所以一般会先给个提示，比如显示一句话 “行为验证加载中” 
	* 如果要在初始化结束后隐藏，在 `captchaObj.onReady()` 中调用即可，这些官方文档都有介绍

```js
// 发起第一步验证
$.ajax({
  url: '${ctx}/captcha',
  type: 'GET',
  dataType: 'json',
  success: function (data) {
    var result = JSON.parse(data.data)

    // 初始化行为验证
    initGeetest({
      gt: result.gt,
      challenge: result.challenge,
      new_captcha: result.new_captcha,
      offline: !result.success,
      product: 'float',
      width: '100%'
    }, function (captchaObj) {
      // 显示行为验证操作元素
      captchaObj.appendTo($memberRegisterPanel.find('.captcha'))

      captchaObj.onReady(function () {
        getCurrentPanel().find('.captcha-tip').hide()
      })

		// .. 执行后续操作
    })
  }
})
```

### 用户前端验证成功后，发起后端核验请求
1. 这一步的操作会在用户进行滑块验证并成功后被调用
2. 如果滑块都没有弄对，则不会发起后端核验
3. 这里发起的验证实际上就是去调用后端的第二步验证接口
	* `mobile` 是传入的手机号
	* 其他的三个参数都是极验插件内部返回的验证结果，用于发送到后端进行二次核验
4. 在请求的回调中可以判断核验结果，并执行后续操作

```js
captchaObj.onSuccess(function () {
  var result = captchaObj.getValidate()
  
	// 发起之前可以先进行一些自定义验证，比如手机号是否填写，格式是否错误
	// 如果自定义验证没有通过，可以使用 captchaObj.reset() 重置验证组件状态

  // 发起二次验证
  $.ajax({
    url: '${ctx}/captcha',
    type: 'POST',
    data: {
      mobile: mobileVal,
      geetest_challenge: result.geetest_challenge,
      geetest_validate: result.geetest_validate,
      geetest_seccode: result.geetest_seccode
    },
    dataType: 'json',
    success: function (data) {
      if (!data.success) {
        // 验证成功后的具体操作
      }
    }
  })
})
```

## 一句话总结
1. 极验的这个行为验证组件接入的方式比较简单，实现的效果也蛮不错的，但我还是认为就一个验证码功能一年收费 6w 起，太贵了，搞不起搞不起，科科