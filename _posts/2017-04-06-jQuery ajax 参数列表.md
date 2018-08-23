---
title: jQuery ajax 参数列表
date: 2017-04-06
author: asing1elife
categories:
 - jquery
 - ajax
tags:
 - jquery
 - ajax
---
> $.ajax() 只有一个参数，该参数为 key-value 对象  
> jQuery 发送的所有 ajax 请求，都是通过调用该 ajax 方法实现  

## 参数列表
1. accepts - Object
	* 用于通知服务器该请求需要接受哪种类型的返回结果
	* 有必要的话，推荐在 $.ajaxSetup() 中设置一次
2. async - Boolean
	* 默认为 true ，即异步
3. beforeSend - Function
	* 请求发送前的回调方法，默认参数为 xhr 和 settings
	* 函数中显示返回 false 将取消本次请求
4. cache - Boolean
	* 请求是否开启缓存，默认为 true
	* 当 dataType 的值等于 script 或 jsonp 时，默认为 false
5. contents - Object
	* 通过字符串或正则表达式配对的对象
	* 根据给定的内容类型，解析请求的返回结果
6. contentType - String
	* 编码类型，相对应与 http 请求头域中的 Content-Type 字段
	* 默认值为 `application/x-www-form-urlencoded; charset=UTF-8`
7. context - Object
	* 设置 ajax 回调函数的上下文
	* 默认上下文为 ajax 请求传入的参数设置对象
	* 例如设置为 document.body ，则所有回调函数将以 body 为上下文
8. converts - Object
	* 数据类型转换器对象
	* 默认为 `{"* text": window.String, "text html": true, "text json": jQuery.parseJSON, "text xml": jQuery.parseXML}`
	* 可写成 `converters: {"json jsonp": function(msg){}}`
9. crossDomain - Boolean
	* 是否跨域，默认为 false
10. data - Object / Array
	* 发送到服务器的数据，默认为 key-value 格式对象
	* 如果将 traditional 设置为 true ，则会将 data 参数的内容序列化，如将 `{a:1, b: 2} -> &a=1&b=2`
11. traditional - Boolean
	* 按照默认方式序列化 data 对象，默认为 false
12. dataFilter - String
	* 处理 XMLHttpRequest 原始响应数据的回调，默认参数为 data 和 type
	* data 是 ajax 返回的原始数据
	* type 是调用 dataType 参数的值
13. dataType - String
	* 预期服务器返回的数据类型 [ xml / html / script / json / jsonp / text ]
	* 设置为 xml 或 text ，返回数据不会被处理
14. success - Function
	* 请求成功后的回调，默认参数为 xhr 和 textStatus 以及 successString
	* 使用 jsonp 请求时，该方法被 `done(function (data, textStatus, jqXHR) {})` 替代
15. error - Function
	* 请求失败时的回调，默认参数为 xhr 和 textStatus [ null / timeout / error / abort / parsererror ] 以及 errorString 
	* 该方法可以设置为一个包含函数的数组，从而保证每个函数一次被调用
	* 跨域脚本或 JSONP 请求时，该方法不会被调用
	* 使用 jsonp 请求时，该方法被 `fail(function (jqXHR, textStatus, errorThrown) {})` 替代
16. complete - Function
	* 请求完成后的回调，默认参数为 xhr 和 textStatus [ success / notmodified / error / timeout / abort / parsererror ]
	* 不论结果是 success 或 error ，都会触发该回调
	* 该方法可以设置为一个包含函数的数组，从而保证每个函数一次被调用
	* 使用 jsonp 请求时，该方法被 `always(function (data or jqXHR, textStatus, jqXHR or errorThrown) {})` 替代
17. global - Boolean
	* 表示是否出发全局 ajax 事件，默认为 true
	* 设置为 false 将导致 ajaxStart / ajaxStop / ajaxSend / ajaxError 不被触发
	* 跨域脚本或 JSONP 请求时，改值默认为 false
18. headers - Object
	* 设置请求头，默认为 key-value 格式对象
	* 该设置会在 beforeSend 被调用之前生效，所以在 beforeSend 中可覆盖该设置
19. isModified - Boolean
	* 只有上次请求响应改变时，才允许请求成功，默认为 false
	* 设置为 true ，若数据自上次请求后没有更改过，则会报错
	* 已 http 包中的 Last-Modified 头信息作为判断依据
20. isLocal - Boolean
	* 将当前运行环境设置为本地，默认为 false
	* 设置为 true ，将影响请求发送时的协议
21. jsonp - String
	* 显示指定 jsonp 请求中的回调函数的名称
	* 若设置为 `jsonp: test` ，则会替换掉默认 callback ，以 `test=?` 传给服务器
	* 若设置为 `jsonp: false`  ，则需要明确设置 jsonpCallback 的值
22. jsonpCallback - String / Function
	* 为 jsonp 请求指定一个回调函数名称或回调函数体，用来取代默认生成的随机函数名
	* 回调函数的返回值就是 jsonpCallback 的结果
23. mimeType - String
	* 设置一个 MIME 类型，用于覆盖 xhr 的 MIME 类型
24. username - String
	* 设置认证请求中的用户名
25. password - String
	* 设置认证请求中的密码
26. processData - Boolean
	* ajax 方法默认会将传入的 data 隐式转换为查询字符串，用于配合默认的 contentType
	* 设置为 false ，则不会进行上述转换操作
27. scriptCharset - String
	* 只能在 `dataType: script` 设置后使用
	* 设置后将影响 <script/> 标签上的 charset 属性
	* 仅推荐在跨域请求中使用
28. statusCode - Object
	* 一组 http 状态码和回调函数对应的 key-value 格式对象，例如 `{404: function () {}}`
	* 可用于根据不同的 http 状态码，执行不同的回调
29. timeout - Number
	* 设置超时时间
30. type - String
	* 可设置为 8 种 http method 之一，不区分大小写
31. url - String
	* ajax 请求的 url 地址
32. xhr - Function
	* 在回调中创建并返回 XMLHttpRequest 对象
33. xhrFields - Object
	* 设置原生的 xhr 对象，默认为 key-value 格式对象