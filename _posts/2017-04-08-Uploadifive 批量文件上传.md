---
title: Uploadifive 批量文件上传
date: 2017-04-08
author: asing1elife
categories:
 - jquery
tags:
 - jquery
 - upload
---
> uploadifive 的前身是 uploadify   
> uploadify 是用 flash 实现的批量文件上传，但目前 flash 技术已被淘汰，取而代之的 HTML5 才是大势所趋  
> 于是就有了采用 HTML5 实现批量文件上传的 uploadifive   

## 官网
[Uploadify](http://www.uploadify.com/) 

## 下载 uploadifive 的插件包，需要注意的是 uploadifive 目前是收费插件

## 引入插件所需 js 、css 文件
```html
<script src="${ctx }/assets/plugins/jquery-2.1.3.min.js" type="text/javascript"></script>
<script src="${ctx }/assets/plugins/uploadifive/jquery.uploadifive.js" type="text/javascript"></script>
<link href="${ctx }/assets/plugins/uploadifive/uploadifive.css" rel="stylesheet" type="text/css"/> 
```

## 定义一个 input 标签用于上传文件
```html
<input id="fileUpload" type="file" />
```

## 调用 uploadifive 初始化方法
```js
$(function () {
	$("#fileUpload").uploadifive({
		"uploadScript": "/api/upload/file"
	});
});
```	

## 属性
1. auto - Boolean - true
	* 当文件被添加到上传队列时，会自动上传
2. buttonClass - String
	* 为上传按钮指定一个类选择器
3. buttonText - String - SELECT FILES
	* 定义显示在按钮上的文本内容
4. uploadScript - String - uploadifive.php
	* 上传文件到服务器的后端请求路径
5. checkScript - String
	* 验证待上传文件是否已存在于文件目录中的后端请求路径
	* 文件不存在则返回 0 ，存在则返回 1
6. dnd - Boolean - true
	* 允许拖拽上传文件
7. fileObjName - String
	* 为待上传的文件指定字段名称供后端接收
8. fileSizeLimit - String
	* 指定单个文件的大小限制
	* 默认单位是 KB ，但可以在数字后明确标识 MB 或 GB ，插件内部会自动转换为 KB
9. fileType - String
	* 指定允许上传的文件类型
	* 支持模糊匹配，例如 `image/*` 只允许上传图片，`image/png` 则只允许上传 png 格式的图片
	* 允许多种类型的文件则需要通过 `|` 进行分隔
	* 模糊匹配采用的格式是 MIME types ，可参见 [Media Types](http://www.iana.org/assignments/media-types/media-types.xhtml)
10. formData - JSON
	* 指定文件上传时需要一起提交的其他数据对象
11. height - Number - 30
	* 指定上传按钮的高度 
12. width - Number - 100
	* 指定上传按钮的宽度
13. itemTemplate - String
	* 指定文件上传队列的 HTML 模版
	* 模版的最外层 div 必须指定类选择器为 `uploadifive-queue-item`
	* `.filename` 显示文件名称
	* `.close` 提供关闭按钮
	* `.fileinfo` 显示文件信息
	* `progress-bar` 显示上传进度
14. method - String - post
	* 指定上传文件的提交方式
	* 取值为 post 或 get
15. multi - Boolean - true
	* 支持多文件上传
16. overrideEvents - JSON
	* 可指定多个插件默认事件中的事件名称，被指定的事件将不会执行
17. queueID - String
	* 指定用于显示上传队列的父级元素
	* 若启用了 dnd 功能，则将文件拖放到该父元素中，会触发文件上传
18. queueSizeLimit - Number
	* 指定上传队列中一次可容纳的最大文件数量
19. removeCompleted - Boolean - false
	* 文件上传完毕后从上传队列中移除
20. simUploadLimit - Number
	* 一次可上传的文件数量
21. truncateLength - Number
	* 指定文件名称的截取长度
	* 设置该值后，文件名称超过该长度将会被截取
22. uploadLimit - Number
	* 指定允许上传的最大文件数量
	* 该值的设定不会影响上传队列中可添加的文件数量

## 事件
### 添加新文件到上传队列中
```js
"onAddQueueItem": function (file) {
	console.log(file.name);
}
```

### 将文件从上传队列中删除
```js
"onCancel": function (file) {
	console.log(file.name);
}
```

### 验证文件是否已存在
```js
"onCheck": function (file, exists) {
	console.log(file.name);
	console.log(exists ? "存在" : "不存在");
}
```

### 清空上传队列
```js
"onClearQueue": function (queue) {
	console.log(queue.css());
}
```

### 销毁 uploadifive 实例
```js
"onDestroy": function () {
	console.log("uploadifive 实例被销毁");
}
```

### 将文件拖拽到上传队列中
```js
"onDrop": function (files, fileDropCount) {
	console.log(files.length);
	console.log("拖拽文件数量：" + fileDropCount)
}
```

### 上传文件出错
```js
"onError": function (errorType) {
	console.log("QUEUE_LIMIT_EXCEEDED: 超过上传队列允许的文件数量");
	console.log("UPLOAD_LIMIT_EXCEEDED: 超过上传文件允许的文件数量");
	console.log("FILE_SIZE_LIMIT_EXCEEDED: 超过上传文件允许的大小限制");
	console.log("FORBIDDEN_FILE_TYPE: 被禁止上传的文件类型");
	console.log("404_FILE_NOT_FOUND: 未找到文件");
}
```

### 浏览器不支持 HTML5 
```js
"onFallback": function () {
	console.loh("浏览器不支持 HTML5");
}
```

### 初始化 uploadifive 完成
```js
"onInit": function () {
	console.log("uploadifive 已初始化完成");
}
```

### 文件上传中
```js
"onProgress": function (file, event) {
	console.log(file.name);
	console.log(event.lengthComputable ? "文件上传进度可计算" : "文件上传进度不可计算");
	console.log("文件已上传" + evenet.loaded + "字节");
	console.log("文件总共" + evenet.total + "字节");
}
```

### 上传队列中的所有文件上传完成
```js
"onQueueComplete": function (uploads) {
	console.log(uploads.attempts);
	console.log("上传成功的文件数量：" + uploads.successful);
	console.log("上传失败的文件数量：" + uploads.errors);
	console.log("上传文件总数量：" + uploads.count);
}
```

### 选择某个文件
```js
"onSelect": function (queue) {
	console.log("被取消的文件数量：" + queue.cancelled);
	console.log("上传队列中的文件数量：" + queue.count);
	console.log("上传错误的文件数量：" + queue.errors);
	console.log("被添加到上传队列中的文件数量：" + queue.queued);
	console.log("被选择的文件数量：" + queue.selected);
}
```

### 文件被上传
```js
"onUpload": function (filesToUpload) {
	console.log("将被上传的文件数量：" + filesToUpload);
}
```

### 每一个文件上传完成
```js
"onUploadComplete": function (file, data) {
	console.log(file.name);
	console.log("后端异步操作返回结果：" + JSON.parse(data));
}
```

### 每一个文件上传开始
```js
"onUploadFile": function (file) {
	console.log(file.name);
}
```

## 方法
### 取消文件上传
* 从上传队列中指定被取消上传的文件
* 取消上传的显示方式，为 true 表示直接取消，为 false 则会有一个淡出隐藏的效果

``` js
$("#uploadFile").uploadifive("cancel", $('.uploadifive-queue-item').first().data('file'), true);
```

### 清空上传队列
```js
$("#uploadFile").uploadifive("clearQueue");
```

### 在控制台输出 uploadifive 详细信息
```js
$("#uploadFile").uploadifive("debug");
```

### 销毁 uploadifive 实例
```js
$("#uploadFile").uploadifive("destroy");
```

### 开始上传文件
* 从上传队列中指定被上传的文件
* 第二个参数未指定则默认上传队列中所有文件 

```js
$("#uploadFile").uploadifive("upload", $('.uploadifive-queue-item').first().data('file'));
```