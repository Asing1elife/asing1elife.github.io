---
title: iframe设置session-timeout
date: 2017-01-20
author: asing1elife
categories:
 - html
 - iframe
tags:
 - html
 - iframe
 - session
---
> 在 iframe 中继承父容器的 session 信息  

## 在iframe需要加载的页面中加入
```js
<%
	response.setHeader("P3P", "CP=CAO PSA OUR");
%>
```
	
## 在需要加载的首页加入
```js
if (top != self) {
	if (top.location != self.location) {
		top.location = self.location;
	}
}
```