---
title: html2canvas+jQuery+SpringMVC 实现网页转图片并保存到服务器
date: 2018-09-14
author: asing1elife
categories:
 - java
 - jquery
 - springmvc
tags:
 - java
 - jquery
 - springmvc
 - html2canvas
---
> 前端使用的是 RequireJS + jQuery   
> 后端使用的是 SpringMVC + MyBatis   

## 涉及资料
1. [html2canvas 官网](http://html2canvas.hertzen.com)
2. [将转换后的图片存储服务器的参考](https://blog.csdn.net/qq_16504067/article/details/77650313)

## 将网页转换为图片
### 下载插件包
1. html2canvas 目前最新版是 **v-1.0.0-aplha.12** ，该版本使用的是 ES6 语法 
2. 但本项目使用的是 jQuery ，并且基于 ES5 的语法，所以引入最新版插件时会一直报错 `Uncaught (in promise)`
3. 对于没有使用的 ES6 语法的项目，建议下载 [2017 年的版本](https://github.com/niklasvh/html2canvas/releases?after=v1.0.0-alpha.3)，本文使用的是 [v-0.5.0-beta4](https://github.com/niklasvh/html2canvas/releases/download/v0.5.0-beta4/html2canvas.min.js)

### 调用插件进行转换
1. 最新版及官网介绍的方法是基于 ES6 的语法结构，对于使用 ES5 语法的项目，需要使用如下方法
2. 方法第一个参数是传入 DOM 元素，而通过 jQuery 获取的 DOM 元素实际上是一个集合，所以需要通过下标指定具体元素后插件才能正常获取
3. 方法第二个参数是传入 options 配置，对于 ES5 语法需要使用 `onrendered: function(canvas) {...}` 来执行回调

```js
html2canvas($diplomaInfoPanel[0], {
  useCORS: true,
  onrendered: function (canvas) {
    // 执行回调
  }
})
```

### 在转换时需要解决图片跨域问题
1. 众所周知 canvas 无法处理跨域图片，那么基于 canvas 实现的 html2canvas 自然也无法处理跨域图片
2. 如果在插件待转换的 DOM 结构中存在跨域图片，则会转换失败并报错
3. 虽然插件本身提供了一些处理跨域的方式，但本文是通过 **后端代理转发图片** 的操作来规避跨域问题的，如下

```html
// 在图片地址直接发起图片请求
<img src="${ctx}/api/diploma/info/${memberDiploma.diploma.hexId}/thumbnail" alt="">
```
```java
// 控制层接收请求
@RequestMapping("/{diplomaId}/thumbnail")
public void thumbnail(HttpServletResponse response, @PathVariable String diplomaId) {
    diplomaInfoService.getThumbnail(response, diplomaId);
}
```
```java
// 逻辑层获取到真实图片后通过流的形式反馈到前端
public void getThumbnail(HttpServletResponse response, String diplomaId) {
    try {
		  // 提供图片完整地址并转换
        BufferedImage read = ImageIO.read(new URL(CommonHelper.buildDiplomaThumbnail(diplomaId)));
        OutputStream outputStream = response.getOutputStream();

        ImageIO.write(read, "png", outputStream);

        read.flush();
        outputStream.flush();
        outputStream.close();
    } catch (IOException e) {
        logger.error("证书图片代理获取失败", e);
    }
}
```

## 将图片保存至服务器
### 前端将图片转换为 BASE64 格式供服务器接收
1. 想要将图片传至后端，需要使用 canvas 的 `canvas.toDataURL('image/jpeg')` 将画布转换为 BASE64 格式的图片
2. BASE64 格式的图片链接对于后端来说就是一个字符串，所以可以直接接收

```js
onrendered: function (canvas) {
  // 画布转图片链接
  var image = canvas.toDataURL('image/jpeg')

  // 自行封装的 ajax 方法
  $.ts.doAction('/api/diploma/info/save', {
    memberDiplomaId: currentMemberDiplomaId,
    image: image
  }, function () {
	  // 异步回调
  }, '', '', '')
}
```

### 接收 BASE64 并重新转换为图片格式
1. 接收前端传入的 BASE64 字符串后，需要通过 `org.apache.commons.codec.binary.Base64` 将其转换为 `byte[]` 格式
	* 需要注意的是字符串中 `data:image/jpeg;base64,` 之后的内容才是有效内容，所以需要在转换时先截取掉这一部分
2. 转换后直接使用 `org.apache.commons.io.FileUtils` 保存图片到指定路径即可

```java
private static final String BASE64_PREFIX = "data:image/jpeg;base64,";

public void saveMemberDiplomaImage(String memberDiplomaId, String image) {
    Base64 base64 = new Base64();
    byte[] bytes = base64.decode(image.substring(BASE64_PREFIX.length()));

    String filePath = SOPConstants.UPLOAD_FILE_ROOT_PATH + Constants.DIPLOMA_MEMBER_FILE_PATH + super.getMember().getHexId() + "/";
    String fileName = Constants.THUMB_FILE_PREFIX + memberDiplomaId + Constants.THUMB_FILE_EXTENSION;

    FileUtils.writeByteArrayToFile(new File(filePath + fileName), bytes);
}
```