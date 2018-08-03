---
title: Java + jQuery 实现阿里云点播
date: 2018-03-11
author: asing1elife
categories:
 - java
 - jquery
 - aliyun
tags:
 - java
 - jquery
 - aliyun
 - player
---
> 阿里云播放器直接在线点播视频以及直播技术，此处记录点播实现过程  

## 官网
[获取阿里云视频播放凭证](https://help.aliyun.com/document_detail/52858.html?spm=a2c4g.11186623.6.692.JMrFcp)
[阿里云Aliplayer播放器](https://player.alicdn.com/aliplayer/tutorial/tutorial.html)

## 准备步骤
1. 创建 [阿里云账号](https://account.aliyun.com/register/register.htm?spm=5176.8142029.388261.21.fdb276f4jqd8UE&oauth_callback=https%3A%2F%2Fwww.aliyun.com%2F%3Fspm%3Da2c4g.11186623.2.3.idyCey)
2. 根据 [流程](https://help.aliyun.com/knowledge_detail/37171.html?spm=a2c4g.11186623.2.4.Y3wd6u) 完成实名认证，以确保可以使用阿里云相应服务
3. 在密钥管理页面获取阿里云访问密钥，AccessKeyId 和 AccessKeySecret 

## 后端相关操作
### 在项目 pom 中引入所需 jar 包
```xml
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>3.2.2</version>
</dependency>
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-vod</artifactId>
    <version>2.7.0</version>
</dependency>
```

### 获取对接需要的数据
1. 将 AccessKeyId 、AccessKeySecret 进行相应存储

### 新建 VideoController 用于接收前端获取视频播放凭证的请求
```java
@Controller
@RequestMapping("/api/video")
public class VideoController extends AbstractBaseController {

    @Autowired
    private VideoServiceImpl videoServiceImpl;

    /**
     * 获取播放凭证
     *
     * @param videoId
     * @return
     */
    @RequestMapping(value = "/auth", method = RequestMethod.POST)
    @ResponseBody
    public ResponseData playAuth(@RequestParam final String videoId) {
        return new SimpleActionHandler(request) {
            @Override
            protected void doHandle(ResponseData responseData) throws Exception {
                responseData.setData(videoServiceImpl.getVideoPlayAuth(videoId));
            }
        }.handle();
    }

}
```

### 新建 VideoServiceImpl 用于和阿里云播放器接口对接
```java
@Service
public class VideoServiceImpl extends AbstractBaseService {
	...
}
```

### 在 VideoServiceImpl 中添加获取客户端的 getClient() 方法
1. `Constants.ALI_ACCESS_KEY_ID` 和 `Constants.ALI_ACCESS_SECRET` 是密钥，成对生成和使用
2. 其他参数信息按照阿里云开发手册说明，均不需要改变

```java
private DefaultAcsClient getClient() {
    // 初始化配置
    DefaultProfile profile = DefaultProfile.getProfile("cn-shanghai", Constants.ALI_ACCESS_KEY_ID, Constants.ALI_ACCESS_SECRET);

    // 获取客户端
    return new DefaultAcsClient(profile);
}
```

### 通过外部传入的视频 id 从客户端获取视频播放凭证
```java
public String getVideoPlayAuth(String videoId) {
    GetVideoPlayAuthRequest request = new GetVideoPlayAuthRequest();
    GetVideoPlayAuthResponse response = null;

    // 播放id
    request.setVideoId(videoId);

    try {
        response = getClient().getAcsResponse(request);
    } catch (ClientException e) {
        logger.error("视频客户端获取失败！", e);
    }

    if (response != null) {
        return response.getPlayAuth();
    }

    return null;
}
```

## 前端相关操作
### 引入播放器所需要的 css/js 文件
1. 以下引入的 js 文件为通用版本，包括了 flash 和 h5 的播放器
2. 如果只想单独引入 flash 或 h5 ，只需要将名称中间加上对应标识即可，例如 `aliplayer-h5-min.js`
3. css 文件为公有版本，无需区分类型

```html
<link rel="stylesheet" href="//g.alicdn.com/de/prismplayer/2.5.1/skins/default/aliplayer-min.css"/>
<script charset="utf-8" type="text/javascript" src="//g.alicdn.com/de/prismplayer/2.5.1/aliplayer-min.js"></script>
```

### 准备待转化为播放器的标签内容
1. 标签中的 `data-id` 是将视频传入到阿里云播放器后端之后返回的一个 vid
2. 该 vid 可以通过 [接口上传](https://help.aliyun.com/document_detail/55407.html?spm=a2c4g.11186623.2.3.DM5WHw) 也可以通过阿里云后端上传，此处不做介绍

```html
<div class="prism-player" id="prismPlayer" data-id="281fc1687cb245658dc5e7462e54bc66"></div>
```

### 初始化视频播放器
1. `$.ts.doAction` 是经过封装后的 ajax 操作
2. `Aliplayer({...})` 则是具体的播放器初始化操作

```javascript
var playerTag = target.find("#prismPlayer");
var videoId = playerTag.data("id");

// 移除文字标识
playerTag.empty();

// 非空验证
if (videoId === undefined) {
    return;
}

$.ts.doAction("/api/video/auth", {
    videoId: videoId
}, function () {
    Aliplayer({
        id: "prismPlayer",
        autoplay: true,
        width: "100%",
        vid: videoId,
        playauth: this.data
    });
}, '', '', '');
```