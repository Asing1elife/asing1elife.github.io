---
title: Javascript 获取 XMLHttpRequest 对象
date: 2017-04-05
author: asing1elife
categories:
 - javascript
 - ajax
tags:
 - javascript
 - ajax
---
> 浏览器通过 XMLHttpRequest 对象实现 Ajax 功能   
> 而 IE6 以下版本只支持通过 ActiveXObject 对象实现 Ajax 功能  
> 以下提供一个全平台兼容获取 Ajax 支持对象的方法  

## 实现方式
```javascript
function getXHR(){
  var xhr = null;

	// 浏览器支持 XMLHttpRequest
  if(window.XMLHttpRequest) {
		xhr = new XMLHttpRequest();
  } else if (window.ActiveXObject) {
		// IE6 以下浏览器只支持 AactiveXObject
    	try {
			// 获取 MSXML3 标准
      	xhr = new ActiveXObject("Msxml2.XMLHTTP");
	    } catch (e) {
  		    try {
				// 备选方案，功能不完善，不推荐使用
      		xhr = new ActiveXObject("Microsoft.XMLHTTP");
		    } catch (e) { 
        		alert("您的浏览器暂不支持Ajax!");
      	}
    	}
  }

  return xhr;
}
```